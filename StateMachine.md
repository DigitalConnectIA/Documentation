# StateMaching
Esta maquina de estados es la encarga de dar el flujo de conversación de cada petición que se le hace al chatbot, consta de varias lambdas, estas mismas son adaptables según los requerimientos de la conversación 
El dato de entrada del la State Machine es el siguiente JSON. En donde sessionID es el identificador del chat ya sea el **ID** de la llamada de voz de Twilio o el **ID** del socket del chat

```json
{"sessionID": "di3Jae7vvHcCFfA=", "mensaje":"hola"}
```

El json de salida de la StateMachine es el siguiente, este formato es importante debido a que el Front-End ya esta configurado para recibir estos datos.

```json
{"respuesta": "hola", "sessionID": "di3Jae7vvHcCFfA=", "intent": "saludo"}
```

## Definición de State Machine
La estructura definida para la State Machine es el siguiente JSON. En este mismo se muestra que las entras y salidas de cada paso son las definidas dentro de las lambdas 

```json
{
  "Comment": "Prueba 01 de flujo de conversacion",
  "StartAt": "ie",
  "States": {
    "ie": {
      "Type": "Task",
      "Resource": "ARN lambda definida desde template SAM",
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
      "Resource": "ARN lambda definida desde template SAM",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "ad"
    },
    "cv": {
      "Type": "Task",
      "Resource": "ARN lambda definida desde template SAM",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "ad"
    },
    "cr": {
      "Type": "Task",
      "Resource": "ARN lambda definida desde template SAM",
      "InputPath": "$",
      "OutputPath": "$",
      "Next": "ad"
    },
    "ad": {
      "Type": "Task",
      "Resource": "ARN lambda definida desde template SAM",
      "InputPath": "$",
      "OutputPath": "$",
      "End": true
    }
  }
}
```


### SFIdentificaEstado
Esta lambda obtiene el JSON de entrada de la State Machine y verifica en la base de datos si existe el **sessionID** de no existir lo almacena y asigna sus datos de **estado** y **n_pregunta**. por último se retorna a la State Machine el JSON que se muestra a continuación

```json
{
  "mensaje": "cual es la hora de londres",
  "estado": "cm",
  "n_pregunta": 0,
  "sessionID": "di3Jae7vvHcCFfA="
}
```

### SFCualquierMensaje
El JSON de entrada es el devuelto por la lambda **SFIdentificaEstado** y devuelve el siguiente.

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

Esta lambda es la encargada de conectar con los diferentes motores, con boto3 invoca la lambda de **Snips** **SemanticDistance** y **GoogleSearch**, un ejemplo de la invocación es con el siguiente código

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

### SFCapturaVenta
La finalidad de esta lambda es para las conversaciones que caen en la intencion de venta de servicios, de igual forma recibe como JSON de entrada la salida de la lambda **SFIdentificaEstado** pero retorna el siguiente JSON

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

En este caso solo se manda traer la lambda de **Snips** y dependiendo de la pregunta en la que va se selecciona la entity que se busca dentro del mensaje un ejemplo es el siguiente códgo

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

### SFCapturaRecluta
Así como las lambdas anteriores, esta también recibe el JSON de **SFIdentificaEstado** y como resultado tiene el siguiente

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

De igual forma, segun la pregunta en la que va, invoca a **Snips** y sus entitys que desean analizar

### SFActualizaDatos
Esta lambda recibe como entrada ya sea el JSON de **SFCapturaRecluta**, **SFCapturaVenta** ó **SFCualquierMensaje**. Su finalidad principal es actualizar la tabla donde se almacena la conversacion del **sessionID** que traer el JSON de entrada, dejando como salida este JSON

```json
{"respuesta": "hola", "sessionID": "di3Jae7vvHcCFfA=", "intent": "saludo"}
```

Este JSON es practicamente la salida de la State Machine
