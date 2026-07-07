El punto de sistemas como Antigravity es albergar agentes de IA en un entorno 'seguro'. Yo no díria seguro luego de haber probado Antigravity y tener que pensar cuidadosamente mis prompts para no romper el codigo de inicio de Windows pero si diria 'comodo'. Un detalle de los agentes antes que vieramos modelos de apps como Claude Cowork y Codex era la absoluta incomodidad de  interactuar con modelos a traves de la CLI, era dificil y requeria un set up y conocimiento inciiales que aumentaban mucho la friccion apra el usuario promedio

ya no es asi, modelos como Google's Antigravity permiten interactuar en lenguaje normal con el modelo agentico y dar instrucciones.
Para mi primer proyecto con antigravity decidi ir por algo extremadamente sencillo: un organizador de archivos automatico usando Python.
## El Organizador 
la tarea desde el punto de vista de un proyecto, era algo de 15 minutos. Escribir un script de Python que corriera automaticamente dentro de una carpeta, identificara archivos por su extension y los organizara en una carpeta correspondiente por tipo cada vez que un archivo o varios entraran a esa carpeta. Nada alucinante pero una herramienta de buena utilidad, en especial para los que trabajamos remoto o lidiamos con abundantes archivos en el dia a dia.

Todas las muestras de trabajo serán en ingles debido a que considero que en lo que respecta a codigo y modelos de inteligencia, el lenguaje ingles es el más indicado para ello por razones de cantidad y calidad del material entrenado (LLMs) y sintaxis (Codigo, en este caso Python)

### Paso 1: Probar capacidades básicas
antes de comenzar quise probar su comportamiento y compliance. Pidio permiso para cada accion como fue configurado y ejecuto lo pedido con un minimo de verbosidad. Sabe explicarse adecuadamente para entregar la idea. Notese que para todo use Gemini 3.5 flash (low) que es un modelo en el que he aprendido a confiar bastante por su capacidad de estructuracion impecable.

inicialmente, pedi que creara unos archivos de pruebo usando una sintaxis basica que mezcla numeros referenciales dentro de strings como X$1Y y algo de logica basica al pedirle 3 archivos de cada tipo con su respectiva nomeclatura. Fue realizado perfectamente.
Luego, que creara las carpetas donde serian guardados, en vez de usar lenguaje largo combine terminos de programacion como arrays. Lo hizo perfectamente.

![[Pasted image 20260707012526.png]]

Manejo de archivos fue perfecto, quise ir mas alla. Internet y Dependencias

Para la siguiente fase, quise probar como manejaria obtener dependencias sencillas, como un library, para trabajar. Para esta automatizacion sencilla, de manera que funcione de forma constante y no correr el script manualmente, *watchdog* seria funcional. Fue pedido y tras una prueba manual confirme que habia instalado las dependencias correctamente.

![[Pasted image 20260707014045.png]]

instalar dependencias correctamente es una tarea que un aficionado o incluso un ingeniero junior puede equivocarse. En este contexto el modelo se aseguro de instalar la ultima release stable, ni muy viejo ni un nightly build asi que por ahi vamos bien.

Acto siguiente quise probar la capacidad de logica avanzada, pero de forma cautelosa.

Le pedi al modelo la elaboracion del script deseado, considerando la posible introduccion de un error comun, que es implementar la automatizacion de forma que solo reaccione a archivos nuevos y jamas organize los existentes, introduciendo un cuello de botella propenso a errores. Le describi explicitamente mi deseo, sin usar terminologia tecnica y lo hizo correctamente. 

![[Pasted image 20260707014517.png]]

El codigo a continuacion muestra un nivel de minuciosidad digno de un programador con experiencia, incluso tomandose la consideracion de añadir automaticamente archivos que deberia ignorar y tomo en consideracion posibles conflictos en el nombre de los archivos, como repeticiones de nombres.

```
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

Las descripciones fueron adecuadas y hasta este punto ha mantenido un perfecto entimiendo del contexto y cosas dichas previamente.

![[Pasted image 20260707014305.png]]

### Tocando startup elements
el resultado fue tan rapido y eficiente que me tome el atrevimiento de enviarlo fuera de la carpeta de trabajo que habia designado para este proyecto. Inicialmente solo le pedi que ubicara el PATH de python.exe lo cual hizo exitosamente (agi viva (?)).
Siguiente, le pedi algo relativamente sencillo pero con enorme potencial de salir mal y generar algun bug molesto cada vez que reinicie mi sistema. Crear un archivo de python autoejecutable en el start up de windows que vigile la carpeta deseada.

para hacer esto de forma segura, tome unas consideraciones extras en el prompt:
1. le pedi tomar pasos de precaucion para evitar que archivos faltantes pudieran corromper el start up
2. le di sugeri una pieza de codigo obtenida de gemini 3.1 pro
con confianza ejecuto el proceso. 
Sin embargo al probarlo, no funciono.


![[Pasted image 20260707015511.png]]

para aquellos con experiencia en python o trabajando en shell notaran un error. Estoy pidiendo un proceso de background en python sin tener una consola abierta. La logica demanda algun salvaguarda que evite la perdida completa del proceso si existe algun error o imprevisto no cierre todo de golpe y se olvide de que existio alguna vez.

Al correrlo, naturalmente dejo de funcionar al primer run, y le he pedido que arregle el problema (luego de describirselo y pedirle que hiciera una prueba en vivo). El resultado:
![[Pasted image 20260707020123.png]]
ejecuto una prueba en vivo, logeo el error, hizo un seguimiento a los archivos originales, determino la falta de un evoltorio headless, lo implemento y arreglo el problema todo en el mismo prompt. El resultado es absurdamente eficiente considerando que esto es el modelo de low de 3.5 Gemini Flash y apenas le tomo 2 minutos.
Ingenieros capaces que no hayan implementado esta sencilla automatizacion antes facilmente podrian pasar varios minutos resolviendo este bug, y luego algo mas implementando el wrapper. El agente apenas se tomo 2 minutos y usando lenguaje sencillo.

# Resultados
El manejo inicial me deja satisfecho y optimista. Lo juzgo como más que adecuado. Especificamente sobre la interfaz de Antigravity, es clara, modesta y elegante. Considero que podrían usar una mejor implementacion en los pasos de seguridad - es posible presionar "Skip" y hacer que el modelo lo interprete como un auto-yes global y ejecute acciones peligrosas. Antigravity es mucho mas liberal con los permisos que le da al modelo que por ejemplo, Claude. 
La herramienta es excelente y recomiendo aprenderla. 

No puedo enfatizar lo suficiente que se debe tener extremo cuidado en los pasos asignados al modelo. Claramente es capaz de afectar los archivos del sistema directamente, y si bien es posible que los guardrails de cada modelo eviten un daño masivo a tu sistema no es buena idea arriesgarse sin necesidad. Prompting cuidadoso!

#antigravity #seed #estado/revision 