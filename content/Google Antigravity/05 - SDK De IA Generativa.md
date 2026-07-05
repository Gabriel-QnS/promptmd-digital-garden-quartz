Para poder integrar IA Generativa a un proyecto, es posible hacer un lanzamiento acelerado con el SDK De Google para esto.
### ¿Que es un SDK?
Son las singlas de Software Development Kit en inglés, o Kit de Desarrollo de Software, es un conjunto de herramientas, bibliotecas, documentación y ejemplos de código que un fabricante proporciona a los desarrolladores para facilitar la creación de aplicaciones dentro de su ecosistema o para utilizar sus servicios específicos. En esencia, es una caja de herramientas completa que evita que debas construir funcionalidades básicas desde cero, permitiéndote integrar hardware o software de terceros de forma eficiente.
#### Si eres programador 
el SDK de IA Generativa no es diferente de importar los archivos de React o módulos de Tailwind para conseguir funciones especializadas sin tener que crearlas tú desde cero.
#### Si no eres programador
Imagina que quieres construir una casa, pero no tienes que fabricar los ladrillos, ni mezclar el cemento, ni forjar las tuberías tú mismo.

Un **SDK** es como recibir un **kit de construcción prefabricado de alta gama** que incluye todas las piezas fundamentales (puertas, ventanas, instalaciones eléctricas) ya diseñadas para encajar perfectamente entre sí. En lugar de pasar años aprendiendo a fundir metales o cocer arcilla, el kit te da todo lo necesario para que tu esfuerzo se centre exclusivamente en el diseño y la arquitectura de la casa, asegurando que, al usar esas piezas estandarizadas, el resultado final sea sólido y funcional.

![[SDK.png]]

Actualmente, el SDK de IA generativa de Google admite [Python](https://pypi.org/project/google-genai/), [Go](https://pkg.go.dev/google.golang.org/genai), [Node.js](https://www.npmjs.com/package/@google/genai), [Java](https://search.maven.org/artifact/com.google.genai/google-genai) y [C#](https://www.nuget.org/packages/Google.GenAI).

Por ejemplo, así es como hablarías con Gemini en Google AI en Python:

```
client = genai.Client(  api_key=your-gemini-api-key)response = client.models.generate_content(  model="gemini-3.5-flash",  contents="Why is the sky blue?")
```

Para hacer lo mismo con Gemini en Agent Platform en Google Cloud, solo debes cambiar la inicialización del cliente, y el resto es igual:

```
client = genai.Client(  vertexai=True,  project=your-google-cloud-project,  location="us-central1")response = client.models.generate_content(  model="gemini-3.5-flash",  contents="Why is the sky blue?")
```