# StateMaching
Esta maquina de estados es la encarga de dar el flujo de conversación de cada petición que se le hace al chatbot, consta de varias lambdas, estas mismas son adaptables según los requerimientos de la conversación 
El dato de entrada del la State Machine es el siguiente JSON. En donde sessionID es el identificador del chat ya sea el `ID` de la llamada de voz de Twilio o el `ID` del socket del chat
```json
{"sessionID": "di3Jae7vvHcCFfA=", "mensaje":"hola"}
```
El json de salida de la StateMachine es el siguiente, este formato es importante debido a que el Front-End ya esta configurado para recibir estos datos.
```json
{"respuesta": "hola", "sessionID": "di3Jae7vvHcCFfA=", "intent": "saludo"}
```
Gráficamente el funcionamiento de la StateMachine queda como se muestra en la siguiente figura.  
  
![Arquitectura StepFunction](https://referencias-documentacion-md.s3-us-west-2.amazonaws.com/StepFunction.jpg)

## Definición de State Machine
La estructura definida para la State Machine es el siguiente JSON. En este mismo se muestra que las entras y salidas de cada paso son las definidas dentro de las lambdas 

```json
{
  "Comment": "Prueba 01 de flujo de conversacion",
  "StartAt": "ie",
  "States": {
    "ie": {
      "Type": "Task",
      "Resource": "ARN lambda SFIdentificaEstado",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "motores"
    },
    "motores": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.estado",
          "StringEquals": "cm",
          "Next": "cm"
        },
        {
          "Variable": "$.estado",
          "StringEquals": "cv",
          "Next": "cv"
        },
        {
          "Variable": "$.estado",
          "StringEquals": "cr",
          "Next": "cr"
        }
      ]
    },
    "cm": {
      "Type": "Task",
      "Resource": "ARN lambda SFCualquierMensaje",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "ad"
    },
    "cv": {
      "Type": "Task",
      "Resource": "ARN lambda SFCapturaVenta",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "ad"
    },
    "cr": {
      "Type": "Task",
      "Resource": "ARN lambda SFCapturaRecluta",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "ad"
    },
    "ad": {
      "Type": "Task",
      "Resource": "ARN lambda SFActualizaDato",
      "InputPath": "$",
      "OutputPath": "$",
      "End": true
    }
  }
}
```

### SFIdentificaEstado
Esta lambda obtiene el JSON de entrada de la State Machine y verifica en la base de datos si existe el `sessionID` de no existir lo almacena y asigna sus datos de `estado` y `n_pregunta`. La variable de entorno que contiene la lambda es:
```
ConversationService = "ARN de lambda ConversationService"
```
Por último se retorna a la State Machine el JSON que se muestra a continuación.
```json
{
  "mensaje": "cual es la hora de londres",
  "estado": "cm",
  "n_pregunta": 0,
  "sessionID": "di3Jae7vvHcCFfA="
}
```

### SFCualquierMensaje
El JSON de entrada es el devuelto por la lambda `SFIdentificaEstado`. Esta lambda es la encargada de conectar con los diferentes motores, con boto3 invoca la lambda de **Snips** **SemanticDistance** y **GoogleSearch**, un ejemplo de la invocación es con el siguiente código
```python
import boto3, os
lambdas = boto3.client('lambda')
lambdaSnips = os.environ['Snips']
j={"bot":"grupovanguardia","message":mensaje,"toggledEntities":{"SYS.TELEFONO":"True","SYS.NOMBRE":"True"}}
respSnips = lambdas.invoke(FunctionName=lambdaSnips,Payload=json.dumps(j))
respSnips = json.load(respSnips['Payload'])
print("snips ",respSnips)
```
En el caso del motor de lex su invocación se hace de la siguiente forma
```python
import boto3, os
lex = boto3.client('lex-runtime')
responseLex = lex.post_text(botName=os.environ['BOT_NAME'],botAlias=os.environ['BOT_ALIAS'],userId='sessionID',inputText=mensaje)
print("lex ",responseLex)
```
Para poder llevar acabo este proceso, es necesario de las siguientes variables de entorno
```
BOT_ALIAS = "Alias del bot lex a usar"
BOT_NAME = "Nombre del bot lex a usar"
GoogleSearch = "ARN lambda GoogleSearch"
NameBotDB = "Nombre de la tabla con las respuestas del BOT a usar"
SemanticDistance = "ARN lambda SemanticDistance"
Snips = "ARN lambda Snips"
```
Por último como salida es este JSON:
```json
{
  "respuesta": "Por el momento no cuento con esa información",
  "mensaje": "cual es la hora de londres",
  "estado": "cm",
  "sessionID": "di3Jae7vvHcCFfA=",
  "n_pregunta": 0,
  "intent": ""
}
```

### SFCapturaVenta
La finalidad de esta lambda es para las conversaciones que caen en la intencion de venta de servicios, de igual forma recibe como JSON de entrada la salida de la lambda `SFIdentificaEstado`
En este caso solo se manda traer la lambda de **Snips** y dependiendo de la pregunta en la que va se selecciona la entidad que se busca dentro del mensaje un ejemplo es el siguiente códgo
```python
import boto3, os
lambdas = boto3.client('lambda')
lambdaSnips = os.environ['Snips']
j={"bot":"grupovanguardia","message":mensaje,"toggledEntities":{"SYS.NOMBRE":"True"}}
respSnips = lambdas.invoke(FunctionName=lambdaSnips,Payload=json.dumps(j))
respSnips = json.load(respSnips['Payload'])
print("snips ",respSnips)
if len(respSnips['slots']) > 0:
  for slot in respSnips['slots']:
    if slot['entity'] == 'SYS.NOMBRE':
      print("Result expr: ",re.search(".(\s+?).", slot['rawValue']))
      if (re.search(".(\s+?).", slot['rawValue']) != None):
        rawValue = slot['rawValue']
        entity = 'SYS.NOMBRE'
```
Esta lambda esta compuesta con las siguientes variables de entorno
```
NameBotDB = "Nombre de la tabla con las respuestas del BOT a usar"
Snips = "ARN lambda Snips"
```
Este proceso arroja un resultado de esta forma
```json
{
  "respuesta":"Dime un numero de telefono a 10 digitos",
  "n_pregunta": 3,
  "mensaje": "juan Sandoval",
  "estado": "cv",
  "sessionID": "di3Jae7vvHcCFfA=",
  "intent":"venta"
}
```

### SFCapturaRecluta
Así como las lambdas anteriores, esta también recibe el JSON de `SFIdentificaEstado`. De igual forma que la lambda `SFCapturaVenta`, segun la pregunta en la que va, invoca a **Snips** y sus entidades que desean analizar. Como resultado tiene el siguiente JSON.
```json
{
  "respuesta":"Dime un numero de telefono a 10 digitos",
  "n_pregunta": 3,
  "mensaje": "juan Sandoval",
  "estado": "cv",
  "sessionID": "di3Jae7vvHcCFfA=",
  "intent":"recluta"
}
```
Esta lambda esta compuesta con las siguientes variables de entorno
```
NameBotDB = "Nombre de la tabla con las respuestas del BOT a usar"
Snips = "ARN lambda Snips"
```

### SFActualizaDatos
Esta lambda recibe como entrada ya sea el JSON de `SFCapturaRecluta`, `SFCapturaVenta` ó `SFCualquierMensaje`. Su finalidad principal es actualizar la tabla donde se almacena la conversacion del `sessionID` que traer el JSON de entrada, dejando como salida este JSON
```json
{"respuesta": "hola", "sessionID": "di3Jae7vvHcCFfA=", "intent": "saludo"}
```
La variable de entorno que contiene esta lambda es:
```
ConversationService = "ARN de lambda ConversationService"
```
