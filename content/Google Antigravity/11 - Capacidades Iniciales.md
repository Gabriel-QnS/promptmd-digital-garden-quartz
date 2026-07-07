El objetivo de sistemas como Antigravity es albergar agentes de IA en un entorno "seguro". Yo no diría seguro, luego de haber probado Antigravity y tener que pensar cuidadosamente mis prompts para no romper el código de inicio de Windows, pero sí diría "cómodo". Un detalle de los agentes antes de que viéramos modelos de aplicaciones como Claude Cowork y Codex era la absoluta incomodidad de interactuar con los modelos a través de la CLI: era difícil y requería una configuración (*setup*) y conocimientos iniciales que aumentaban mucho la fricción para el usuario promedio.

Ya no es así; modelos como Antigravity de Google permiten interactuar en lenguaje normal con el modelo agéntico y darle instrucciones.
Para mi primer proyecto con Antigravity, decidí ir por algo extremadamente sencillo: un organizador de archivos automático usando Python.
## El organizador
La tarea, desde el punto de vista de un proyecto, era algo de 15 minutos: escribir un script de Python que corriera automáticamente dentro de una carpeta, identificara archivos por su extensión y los organizara en la carpeta correspondiente por tipo cada vez que un archivo o varios entraran a esa carpeta. Nada alucinante, pero es una herramienta de gran utilidad, en especial para los que trabajamos de forma remota o lidiamos con abundantes archivos en el día a día.

Todas las muestras de trabajo serán en inglés, debido a que considero que en lo que respecta a código y modelos de inteligencia, el idioma inglés es el más indicado por razones de cantidad y calidad del material de entrenamiento (LLMs) y sintaxis (código, en este caso Python).

### Paso 1: Probar capacidades básicas
Antes de comenzar, quise probar su comportamiento y conformidad (*compliance*). Pidió permiso para cada acción, tal como fue configurado, y ejecutó lo pedido con un mínimo de verbosidad. Sabe explicarse adecuadamente para transmitir la idea. Nótese que para todo usé Gemini 3.5 Flash (Low), que es un modelo en el que he aprendido a confiar bastante por su capacidad de estructuración impecable.

Inicialmente, pedí que creara unos archivos de prueba usando una sintaxis básica que mezcla números referenciales dentro de strings como X$1Y y algo de lógica básica al pedirle 3 archivos de cada tipo con su respectiva nomenclatura. Fue realizado perfectamente.
Luego, le pedí que creara las carpetas donde serían guardados; en vez de usar un lenguaje largo, combiné términos de programación como arrays. Lo hizo perfectamente.

![[Pasted image 20260707012526.png]]

El manejo de archivos fue perfecto, así que quise ir más allá: Internet y dependencias.

Para la siguiente fase, quise probar cómo manejaría la obtención de dependencias sencillas, como una biblioteca (library), para trabajar. Para esta automatización sencilla, con el fin de que funcione de forma constante y sin tener que ejecutar el script manualmente, *watchdog* sería funcional. Se lo pedí y, tras una prueba manual, confirmé que había instalado las dependencias correctamente.

![[Pasted image 20260707014045.png]]

Instalar dependencias correctamente es una tarea en la que un aficionado o incluso un ingeniero junior se puede equivocar. En este contexto, el modelo se aseguró de instalar la última versión estable (stable release), ni muy vieja ni una compilación nocturna (nightly build), así que por ahí vamos bien.

Acto seguido, quise probar la capacidad de lógica avanzada, pero de forma cautelosa.

Le pedí al modelo la elaboración del script deseado, considerando la posible introducción de un error común: implementar la automatización de forma que solo reaccione a archivos nuevos y jamás organice los existentes, lo que introduciría un cuello de botella propenso a errores. Le describí explícitamente mi deseo sin usar terminología técnica, y lo hizo correctamente.

![[Pasted image 20260707014517.png]]

El código a continuación muestra un nivel de minuciosidad digno de un programador con experiencia, incluso tomándose la consideración de añadir automáticamente archivos que debería ignorar y tomando en cuenta posibles conflictos en el nombre de los archivos, como repeticiones de nombres.

```python
import os
import sys
import shutil
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# Mapping of file extensions to their target directories
EXTENSION_MAP = {
    '.pdf': 'pdf files',
    '.txt': 'txt files',
    '.jpg': 'jpg files',
    '.jpeg': 'jpg files'
}

# Files to ignore (e.g., the script itself, requirements, etc.)
IGNORED_FILES = {
    'auto_organizer.py',
    'requirements.txt',
    'generate.py',
    'organizer.log'
}

# Define root folder of operations
root_directory = os.path.dirname(os.path.abspath(__file__))
log_file_path = os.path.join(root_directory, "organizer.log")

# Setup robust logging that handles headless execution
class Logger:
    def __init__(self, filepath):
        self.terminal = sys.stdout
        try:
            self.log = open(filepath, "a", encoding="utf-8")
        except Exception:
            self.log = None

    def write(self, message):
        if self.terminal:
            try:
                self.terminal.write(message)
            except Exception:
                pass
        if self.log:
            try:
                self.log.write(message)
                self.log.flush()
            except Exception:
                pass

    def flush(self):
        if self.terminal:
            try:
                self.terminal.flush()
            except Exception:
                pass
        if self.log:
            try:
                self.log.flush()
            except Exception:
                pass

sys.stdout = Logger(log_file_path)
sys.stderr = Logger(log_file_path)

def get_destination_folder(filename):
    _, ext = os.path.splitext(filename.lower())
    if ext in EXTENSION_MAP:
        return EXTENSION_MAP[ext]
    return None

def move_file(src_path, dest_dir):
    try:
        if not os.path.exists(dest_dir):
            os.makedirs(dest_dir, exist_ok=True)
            
        filename = os.path.basename(src_path)
        dest_path = os.path.join(dest_dir, filename)
        
        # Handle file name collisions
        if os.path.exists(dest_path):
            base, ext = os.path.splitext(filename)
            counter = 1
            while os.path.exists(dest_path):
                dest_path = os.path.join(dest_dir, f"{base}_{counter}{ext}")
                counter += 1
                
        shutil.move(src_path, dest_path)
        print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Moved: {filename} -> {os.path.basename(os.path.dirname(dest_path))}/")
    except Exception as e:
        print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Error moving {src_path}: {e}")

def organize_all(root_dir):
    print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Performing initial sweep to sort existing files...")
    for item in os.listdir(root_dir):
        item_path = os.path.join(root_dir, item)
        if os.path.isfile(item_path) and item not in IGNORED_FILES:
            dest_folder_name = get_destination_folder(item)
            if dest_folder_name:
                dest_dir = os.path.join(root_dir, dest_folder_name)
                move_file(item_path, dest_dir)
    print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Initial sweep complete.")

class FileOrganizerHandler(FileSystemEventHandler):
    def __init__(self, root_dir):
        super().__init__()
        self.root_dir = os.path.abspath(root_dir)

    def on_created(self, event):
        if event.is_directory:
            return
        
        src_path = os.path.abspath(event.src_path)
        filename = os.path.basename(src_path)
        
        # Skip ignored files
        if filename in IGNORED_FILES:
            return
            
        # Check if the file is in the root directory (not already inside a subdirectory)
        if os.path.dirname(src_path) == self.root_dir:
            # Add a small delay to make sure the file is completely written before moving
            time.sleep(0.5)
            dest_folder_name = get_destination_folder(filename)
            if dest_folder_name:
                dest_dir = os.path.join(self.root_dir, dest_folder_name)
                move_file(src_path, dest_dir)

if __name__ == "__main__":
    # Sort existing files on startup
    organize_all(root_directory)
    
    # Setup Watchdog observer
    event_handler = FileOrganizerHandler(root_directory)
    observer = Observer()
    observer.schedule(event_handler, path=root_directory, recursive=False)
    observer.start()
    
    print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Monitoring folder: {root_directory}")
    print("Press Ctrl+C to stop.")
    
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
        print(f"\n[{time.strftime('%Y-%m-%d %H:%M:%S')}] Stopping organizer...")
    observer.join()
```

Las descripciones fueron adecuadas y hasta este punto ha mantenido un perfecto entendimiento del contexto y de lo dicho previamente.

![[Pasted image 20260707014305.png]]

### Tocando elementos de inicio (startup elements)
El resultado fue tan rápido y eficiente que me tomé el atrevimiento de probarlo fuera de la carpeta de trabajo que había designado para este proyecto. Inicialmente solo le pedí que ubicara la ruta (PATH) de `python.exe`, lo cual hizo exitosamente (¿AGI viva?).
Después, le pedí algo relativamente sencillo pero con un enorme potencial de salir mal y generar algún bug molesto cada vez que reinicie mi sistema: crear un archivo de Python autoejecutable en el inicio (startup) de Windows que vigile la carpeta deseada.

Para hacer esto de forma segura, tomé algunas consideraciones adicionales en el prompt:
1. Le pedí tomar medidas de precaución para evitar que archivos faltantes pudieran corromper el inicio (startup).
2. Le sugerí una pieza de código obtenida de Gemini 3.1 Pro.
Con confianza, ejecutó el proceso. Sin embargo, al probarlo, no funcionó.


![[Pasted image 20260707015511.png]]

Para aquellos con experiencia en Python o trabajando en la terminal (shell), notarán un error: estoy pidiendo un proceso en segundo plano (background) en Python sin tener una consola abierta. La lógica demanda alguna salvaguarda que evite la pérdida completa del proceso si existe algún error o imprevisto, en lugar de cerrar todo de golpe y olvidar que existió alguna vez.

Al correrlo, naturalmente dejó de funcionar en la primera ejecución, y le pedí que arreglara el problema (luego de describírselo y pedirle que hiciera una prueba en vivo). El resultado:
![[Pasted image 20260707020123.png]]
Ejecutó una prueba en vivo, registró el error, hizo un seguimiento a los archivos originales, determinó la falta de un contenedor headless (wrapper), lo implementó y arregló el problema, todo en el mismo prompt. El resultado es absurdamente eficiente considerando que este es el modelo low de Gemini 3.5 Flash y apenas le tomó 2 minutos.
Ingenieros capaces que no hayan implementado esta sencilla automatización antes podrían pasar varios minutos resolviendo este bug, y luego algo más implementando el wrapper. El agente apenas se tomó 2 minutos y usando un lenguaje sencillo.

# Resultados
El manejo inicial me deja satisfecho y optimista. Lo considero más que adecuado. Específicamente sobre la interfaz de Antigravity, es clara, modesta y elegante. Considero que podrían usar una mejor implementación en los pasos de seguridad: es posible presionar "Skip" y hacer que el modelo lo interprete como un "sí" automático global y ejecute acciones peligrosas. Antigravity es mucho más liberal con los permisos que le da al modelo en comparación con, por ejemplo, Claude. 

La herramienta es excelente y recomiendo aprender a usarla. 

No puedo enfatizar lo suficiente que se debe tener extremo cuidado con los pasos asignados al modelo. Claramente es capaz de afectar los archivos del sistema directamente y, si bien es posible que los límites de seguridad (guardrails) de cada modelo eviten un daño masivo a tu sistema, no es buena idea arriesgarse sin necesidad. ¡Prompting cuidadoso!

Luego de todo decidí escribir esto en mi galería, lo escribí sin fijarme mucho en la gramática, y simplemente le pedí a antigravity que se encarga de hacer un check de gramática básico y aplicarlo, en español latino. ¡Maravillas!

#Antigravity  #estado/revision 
