# API's
Estas API's son usadas en el Front-End de Amplify, cada una será especificada y detalla para conocer su funcionamiento, hasta el momento se tienen solo dos y son
* PointAccess
* socket_chatbot

## PointAccess
Esta API es la central y esta compuesta por la siguientes rutas
* /clients
* /conversation
* /convertest
* /engine
* /import
* /start
* /token
* /user
* /{folder}/{item}

### /clients
Esta ruta tiene la integracion con la lambda llamada ***ClientService***. Por el momento la configuracion de esta lambda solo requiere de 
Definicion de creacion de cliente, ```email``` -> requerido
```json
{
    "operation": "create",
    "payload": {
        "Item": {
            "name": "Jose Luis",
            "lastName": "Palillero Huerta",
            "email": "jlpalillero1@digitialconnect.com.mx",
            "phone": "+5222233232223"
        }
    }
}
```
Definicion de getItem para los clientes, ```email``` -> requerido
```json
{
    "operation": "getItem",
    "payload": {
        "Item": {
            "email": "jlpalillero1@digitialconnect.com.mx"
        }
    }
}
```
Definicion de checkemail para clientes
```json
{
    "operation": "checkemail",
    "payload": {
        "Item": {
            "email": "jlpalillero1@digitialconnect.com.mx"
        }
    }
} 
```
Definicion de update para clientes
```json
{
    "operation": "update",
    "payload": {
        "Item": {
          "id": 1,
          "email": "jlpalillero@digitialconnect.com.mx",
          "lastName": "Palillero Huerta",
          "name": "Jose Luis",
          "phone": "+5222233232223",
          "rol": "portal"
        }
    }
}
```
Definicion de delete para clientes
```json
{
    "operation": "delete",
    "payload": {
        "Item": {
            "email": "jlpalillero@digitialconnect.com.mx"
        }
    }
}
```


### /engine
Esta ruta debe recibir la informacion en metodo **GET** y formato ```text/xml``` La lambda controladora de esta ruta se llama **TwilioEngine** esta debe recibir la variable ```event['queryStringParameters']['SpeechResult']``` que es el resultado de la transcripcion de Twilio y ```event['queryStringParameters']['CallSid']``` que es la **sessionID** de la State Machine la cual se manda traer de la misma forma que en la **API Sockets**.
Mediante la libreria Twilio que utiliza la lambda, la respuesta debe ser un JSON con el siguiente formato

```json
{
  "statusCode": 200,
  "headers": {
    "Content-Type": "text/xml"
  },
  "body": "<?xml version='1.0' encoding='UTF-8'?><Response><Say voice='Polly.Lupe-Neural'>Por el momento no cuento con esa informaci&#243;n</Say><Gather action='https://yshohmubf1.execute-api.us-west-2.amazonaws.com/beta/engine' input='speech' language='es-MX' method='GET' speechTimeout='auto' /></Response>"
}
```

### /start
Esta ruta debe recibir la peticion en metodo **GET** y formato ```text/xml``` La lambda controladora de esta ruta se llama **TwilioStart** esta no recibe variable y solo ejecuta la respuesta rapida para iniciar una conversación de voz.
Mediante la libreria Twilio que utiliza la lambda, la respuesta debe ser un XML con el siguiente formato

```xml
<?xml version='1.0' encoding='UTF-8'?>
<Response>
  <Say voice='Polly.Lupe-Neural'>Hola</Say>
  <Gather action='https://yshohmubf1.execute-api.us-west-2.amazonaws.com/beta/engine'
    input='speech' language='es-MX' method='GET' speechTimeout='auto'>
</Response>
```

### Ruta /token
Esta ruta debe recibir la peticion en metodo **GET** sin datos de envío. La lambda controladora de esta ruta se llama **TwilioToken** esta no recibe variable y solo ejecuta la respuesta que es un token de Twilio para poder generar llamdas desde el Fron-End. Su respuesta es 

```json
{
  "body": "token_de_twilio"
}
```

### Ruta /convertest
Es controlada por una lambda llamada **ConversationTests**, consiste en almacenar en una base de datos los mensajes de prueba junto a la evaluación que le da el agente, el JSON que debe recibir es el siguiente.

```json
{
  "NameBot": "grupovanguardia",
  "NewData": {
    "bot": "hola buen dia hablas a grupo vanguardia",
    "datetime": "09/12/2021:14:00:00",
    "type": "vp",
    "user": "hola"
}
```


### Ruta /conversations
Definicion de creacion de una conversacion, ```idClient``` -> referencia al ID de cliente
```json
{
    "operation": "create",
    "payload": {
        "Item": {
          "transcription": {
            "messageIn": "data",
            ...
          },
          "states": {},
          "idClient": null, 
          "intent": null,
          "lastState": null,
          "lasQuestion": null,
        }
    }
}
```
Definicion de getItem para la conversacion
```json
{
    "operation": "getItem",
    "payload": {
        "Item": {
            "sessionId": "cb00463f-7166-11eb-9998-c727838c173"
        }
    }
}
```
Definicion de update para la conversacion, ```sessionId``` -> requerido, ```idClient``` -> referencia al ID del cliente
```json
{
    "operation": "update",
    "payload": {
        "Item": {
          "sessionId": "cb00463f-7166-11eb-9998-c727838c1732",
          "transcription": {
            "messageIn": "data",
            ...
          },
          "states": {},
          "idClient": null,
          "intent": null,
          "lastState": null,
          "lasQuestion": null,
        }
    }
}
```
Definicion de delete para la conversación
```json
{
    "operation": "delete",
    "payload": {
        "Item": {
            "sessionId": "cb00463f-7166-11eb-9998-c727838c173"
        }
    }
}
```

### Ruta /user
Definicion de creacion de usuario, ```email``` -> requerido, ```password``` -> requerido
```json
{
    "operation": "create",
    "payload": {
        "Item": {
            "name": "Jose Luis",
            "lastName": "Palillero Huerta",
            "email": "jlpalillero1@digitialconnect.com.mx",
            "phone": "+5222233232223",
            "rol": "portal",
            "password": "1234"
        }
    }
}
```
Definicion de getItem para los usuarios, ```email``` -> requerido
```json
{
    "operation": "getItem",
    "payload": {
        "Item": {
            "email": "jlpalillero1@digitialconnect.com.mx"
        }
    }
}
```
Definicion de login para usuarios
```json
{
    "operation": "login",
    "payload": {
        "Item": {
            "email": "jlpalillero1@digitialconnect.com.mx",
            "password": "1234"
        }
    }
} 
```
Definicion de update para usuarios
```json
{
    "operation": "update",
    "payload": {
        "Item": {
          "id": 1,
          "email": "jlpalillero@digitialconnect.com.mx",
          "lastName": "Palillero Huerta",
          "name": "Jose Luis",
          "phone": "+5222233232223",
          "rol": "portal"
        }
    }
}
```
Definicion de delete para usuarios
```json
{
    "operation": "delete",
    "payload": {
        "Item": {
            "email": "jlpalillero@digitialconnect.com.mx"
        }
    }
}
```



### Ruta /import
EL controlador de esta ruta es la lambda con nombre Lex, esta genera un lex mediante la extracción de un zip en s3 el cual debió importarse previamente. La estructura del del zip debe ser de la siguiente forma ```grupovanguardia.zip/botgrupovanguardia.json```. Como respuesta devuelve el siguiente JSON

```json
{
  "NombreBot":"botgrupovanguardia",
  "TipoRecurso":"BOT",
  "Estado":"BUILD"
}
```
La petición debe ser POST con un json como se muestra a continuacion. Donde bucket es el nombre la cubeta y zip el archivo con los datos del bot a entrenar, este previamente se debio enviar en la ruta raiz de esta misma api es decir: ```https://APILex.amazonws/beta/{nombreBucket}/{nombreArchivo}```

```json
{
  "bucket":"datoslex",
  "zip":"grupovanguardiaLex.zip"
}
```

## socket_chatbot
Esta API consta de tres rutas **$connect**, **$disconnect** y **mensaje**. Cada una cuenta con su respectiva lambda con los siguientes nombres **SocketConnect**, **SocketDisconnect** y **SocketMensaje**

### SocketConnect
El dato importante de entrada para esta lambda es el ID de conexión este viene dentro de la siguiente variable ```event['requestContext']['connectionId']```. Este dato debe seer enviado a la base de datos para identificar una nueva conexión y por lo tanto una nueva conversación

### SocketMensaje
Esta lambda es la encargada de procesar los datos de un socket cuando ya está conectado. El JSON que se debe enviar a la ruta **mensaje** es el siguiente

```json
{"action": "mensaje", "datos":"hola"}
```

Por lo tanto la lambda recibe los siquientes datos ```event['requestContext']['connectionId']``` es el ID con el que se identifica la conexión ```event['body'])['datos']``` es el mensaje que va hacia la State Machine. Para invocar la State Machine se utiliza el siguiente código

```python
import boto3, os
sf = boto3.client('stepfunctions')
sf_data = "{\"sessionID\":\""+event['requestContext']['connectionId']+"\",\"mensaje\":\""+json.loads(event['body'])['datos']+"\"}"
sf_respuesta = sf.start_sync_execution(
  stateMachineArn=os.environ['STATE_MACHINE'],
  input=str(sf_data))
sf_data_output=json.loads(sf_respuesta['output'])
print("Result SF ",sf_data_output)
```

Como los socket no tienen **return** para poder regresar la respuesta de la State Machine se genera el siguiente codigo

```python
import boto3, os
api_gateway = boto3.client('apigatewaymanagementapi',endpoint_url = "https://" + event["requestContext"]["domainName"] + "/" + event["requestContext"]["stage"])
api_gateway.post_to_connection(
  Data=sf_respuesta['output'],
  ConnectionId=event['requestContext']['connectionId']
)
```
De esta forma aseguramos regresar el mensaje al mismo socket que hizo la petición
### SocketDisconnect
Esta lambda responde a la ruta **$disconnect**, por el momento no se tiene ninguna funcionalidad solo genera la desconexion segura de un socket