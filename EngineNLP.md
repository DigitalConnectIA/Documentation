# EngineNLP
Para el funcionamiento del chatbot se requiere de varios motores, cada uno recibe diferentes JSON. Aquí solo se documenta el proceso de consulta de cada una de los motores que funcionan dentro de lambdas. En cada motor se especifica su funcionamiento y sus requerimientos. Los motores son:
* Motor Snips
* Motor SemanticDistance
* Motor GoogleSearch
* Motor EntidadesPLN

## Motor Snips
No cuenta con API, su invocación debe ser con boto3 y depende de la layer `LayerSnips`, así como también se le debe especificar las siguientes variables de entorno.
```
ENTIDADESPLN = "ARN de lambda EntidadesPLN"
MODEL_BUCKET_NAME = "Nombre del bucket del modelo para snips"
```
Para su invocación es necesario enviarle el siguiente JSON:

```json
{  
  "bot":"nombreEspecificoDeBot",  
  "message":"frase a categorizar",  
  "toggledEntities":{
      "SYS.TIEMPO":"True",
      "SYS.MONEDA":"True",
      "SYS.CP":"True",
      "SYS.TELEFONO":"True",
      "SYS.FECHA":"True",
      "SYS.PORCENTAJE":"True",
      "SYS.CORREO":"True",
      "SYS.URL":"True",
      "SYS.NUMERO.VUELO":"True",
      "SYS.DURACION":"True",
      "SYS.NOMBRE":"True",
    }
}
```
Referente al uso de entidades seleccionadas, hay 3 modos de usarla.
* Omitir `toggleEntities` asume todas los valores como "True"
* Incluir `toggleEntities` pero omitir entidades en específico asume el valor de dichas entidades como "False" (Que no se incluirán en la respuesta de la API)
* Incluir `toggleEntities` con todas las entidades, pero específicando unas como "True" y otras como "False"

Respuesta:
```json
{
  "statusCode": 200,
  "body": {
    "input": "tienes vacantes",
    "intent": {
      "intentName": "sindato",
      "probability": 0.2696685356480976
    },
    "slots": []
  }
}
```

## Motor EntidadesPLN
Este motor se invoca cuando se hace una peticion al motor de `Snips` el cual de manera automático le manda una petición, debido a que es encargado de identificar entidades, el JSON que recibe es el siguiente
```json
{
  "bot": "grupovanguardia",
  "message": "cual es la hora de londres",
  "toggledEntities": {
    "SYS.TELEFONO": "True", 
    "SYS.NOMBRE": "True"
  },
  "response": {
    "input": "cual es la hora de londres",
    "intent": {
      "intentName": "sindato",
      "probability": 0.24092101964766058
    },
    "slots": []
  }
}

```

## Motor SemanticDistance
Esta lambda en su invocación debe llevar el siguiente JSON. Donde `requestText` es el mensaje que se desea medir la distancia y `responseTex` es un vector con los lexemas y sus respectivos pesos. Tambien depende de una layer que lleva como nombre `LayerDistance`

```json
{
  "requestText": "Cuál es la hora de Londres",
  "responseText": [
    ["hol", 3.0], ["buen", 1.0], ["dias", 1.0], ["noch", 1.0], ["mi", 1.0],["me", 1.0], ["llam", 1.0], ["nombr", 1.0], ["present", 1.0], ["soy", 1.0], ["onda", 4.0], ["tranz", 1.0]
  ]
}
```
Como resultado de la medicion se regresa este JSON. El cual indica la distancia en un valor de 0.0 a 1.0 donde 1.0 es una distancia grande y no son oraciones parecidas

```json
{
  "distance": 1.0
}
```

## Motor GoogleSearch
La Lambda de este motor debe recibir el siguiente JSON. Y la layer que utiliza lleva como nombre `LayerGoogle`

```json
{
  "text": "cual es la hora de londres"
}
```
como respuesta envía
```json
{
  "answer": "Por el momento no cuento con esa información"
}
```
