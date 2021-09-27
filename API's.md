# API's
Estas API's son usadas en el Front-End de Amplify, cada una será especificada y detalla para conocer su funcionamiento, hasta el momento se tienen solo dos y son
* **PointAccess**
* **socket_chatbot**

## PointAccess
Esta API es la central, su configuración en el Template SAM ya incluye la activación de cors y el método de respuesta de cada una de las siguientes rutas.
* /clients
* /conversation
* /convertest
* /engine
* /import
* /metrics
* /reports
* /SemanticDistanceTrainer
* /SnipsTrainer
* /start
* /token
* /user
* /{folder}/{item}

### /clients
Esta ruta tiene la integracion con la lambda llamada `ClientService`. La configuracion de esta lambda solo requiere de las siguientes variables de entorno:
```
cluster_arn_aurora = "ARN del cluster"
secret_arn_aurora = "aquí Jose Luis"
```
La peticion a esta ruta es por el método `POST` y puede recibir cualquiera de los siguientes JSON.  
Definicion de creacion del cliente, ```email``` -> requerido
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

### /conversations
La integración de esta ruta está compuesta por la lambda `ConversationService`. La configuracion de esta lambda solo requiere de las siguientes variables de entorno:
```
cluster_arn_aurora = "ARN del cluster"
secret_arn_aurora = "aquí Jose Luis"
```
La peticion a esta ruta es por el método `POST` y puede recibir cualquiera de los siguientes JSON.  
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

### /convertest
Tienen una integración con la lambda `ConversationTests`, consiste en almacenar en una base de datos los mensajes de prueba junto a la evaluación que le da el agente. Contiene una sola variable de entorno que es:
```
DBTableName = "nombre de la tabla DynamoDB"
```
El JSON que recibe mediante el método `POST` es el siguiente.
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

### /engine
Esta ruta tiene la integracion con la lambda `TwilioEngine` la cual depende del layer llamado `LayerTwilio` y contiene las siguientes variables de entorno
```
ConversationService = "ARN lambda ConversationService"
MAQUINA_ESTADOS = "ARN State Machine chatbot"
TWILIO_CONVERSACION = "URL API PointAccess ruta /engine"
```
La ruta recibe la informacion en metodo `GET` y formato ```text/xml``` 
```json
{
  "statusCode": 200,
  "headers": {
    "Content-Type": "text/xml"
  },
  "body": "<?xml version='1.0' encoding='UTF-8'?><Response><Say voice='Polly.Lupe-Neural'>Por el momento no cuento con esa informaci&#243;n</Say><Gather action='https://yshohmubf1.execute-api.us-west-2.amazonaws.com/beta/engine' input='speech' language='es-MX' method='GET' speechTimeout='auto' /></Response>"
}
```

### /import
La integración de esta ruta es la lambda `Lex`, esta genera un bot Lex mediante la extracción de un zip en s3 el cual debió importarse previamente en la ruta ```/{folder}/{item}```.
Su peticion es con el método `POST` y en el body se inserta el nombre del bucket y el archivo que contiene los datos del nuevo bot
```json
{
  "bucket":"datoslex",
  "zip":"grupovanguardiaLex.zip"
}
```

### /metrics
Esta ruta devuelve las metricas para mostrar al front y sus json son los siguientes
```json
{
    "operation": "getCounterQuestionsByConversationAndDates",
    "payload": {
        "Item": {
            "dateStart": "2021-05-01 00:00:00",
            "dateEnd": "2021-05-31 00:00:00"
        }
    }
}
```
otra peticion:
```json
{
    "operation": "getUsersToday",
    "payload": {
        "Item": {}
    }
}
```
Otra peticion: 
```json
{
    "operation": "getAllUsersToday",
    "payload": {
        "Item": {}
    }
}
```

### /reports
La integración que tiene esta ruta es con la lambda `ReportsService`. La configuracion de esta lambda solo requiere de las siguientes variables de entorno:
```
cluster_arn_aurora = "ARN del cluster"
secret_arn_aurora = "aquí Jose Luis"
```
La peticion a esta ruta es por el método `POST` y puede recibir cualquiera de los siguientes JSON  
Definicion de creacion de reporte, todos los campos son requeridos
```json
{
    "operation": "create",
    "payload": {
        "Item": {
            "agent": "Jose Luis",
            "comment": "No le han llamado para su cotizacion",
            "priorityAttention": true,
            "processStatus": "pendiente"
        }
    }
}
```
Definicion de fullGet para extraer todos los reportes
```json
{
    "operation": "fullGet",
    "payload": {
        "Item": {}
    }
}
```
Definicion de la obtencion de reportes por rango de fechas. Fechas requeridas
```json
{
    "operation": "getReportByDate",
    "payload": {
        "Item": {
            "dateStart": "2021-06-15 00:00:00",
            "dateEnd": "2021-06-15 23:59:59"
        }
    }
}
```
Definicion de la obtencion de reportes por status del reporte
```json
{
    "operation": "getReportByStatus",
    "payload": {
        "Item": {
            "status": "activo"
        }
    }
}
```
Definicion de la obtencion de reportes por agente
```json
{
    "operation": "getReportByAgent",
    "payload": {
        "Item": {
            "agent": "activo"
        }
    }
}
```
Definicion para la actualiacion del reporte. reportId requerido
```json
{
  "operation": "update",
  "payload": {
      "Item": {
          "reportId": "85cdc38ade",
          "agent": "sandoval",
          "comment": "No le han llamado para su cotizacion. Llamada realizada",
          "priorityAttention": true,
          "processStatus": "solucionado"
      }
  }
}
```
Definicion para eliminar el reporte. reportId requerido
```json
{
    "operation": "delete",
    "payload": {
        "Item": {
            "reportId": "85cdc38ade"
        }
    }
}
```

### /SemanticDistanceTrainer
Son peticiones mediante post con los siguientes json disponibles:
```json
{
    "operation": "get",
    "payload": {}
}
```
El siguiente json agrega lexemas a una intent:
```json
{
    "operation": "add",
    "payload": {
        "intent": "ventas",
        "lexemas": "trabajo vacante empleo"
    }
}
```

### /SnipsTrainer
El siguiente json selecciona el nombre del motor snips que desea reentrenar:
```json
{
    "bot": "grupovanguardia"
}
```

### /start
La lambda en la integración de esta ruta se llama `TwilioStart` esta no recibe variable y solo ejecuta la respuesta rapida para iniciar una conversación de voz. Pero si requiere de la layer `LayerTwilio` y de la siguiente variable de entorno
```
TWILIO_CONVERSACION = "URL API PointAccess ruta /engine"
```
Esta ruta debe recibir la peticion en metodo `GET` y formato ```text/xml``` 

```xml
<?xml version='1.0' encoding='UTF-8'?>
<Response>
  <Say voice='Polly.Lupe-Neural'>Hola</Say>
  <Gather action='https://yshohmubf1.execute-api.us-west-2.amazonaws.com/beta/engine'
    input='speech' language='es-MX' method='GET' speechTimeout='auto'>
</Response>
```

### /token
La lambda en la integración de esta ruta se llama `TwilioToken` esta no recibe variable y solo ejecuta la respuesta que es un token de Twilio para poder generar llamdas desde el Fron-End. Requiere de la layer `LayerTwilio` y de la siguientes variables de entorno
```
TWILIO_ACCOUNT_SID = "Lo da Twilio al crear cuenta"
TWILIO_AUTH_TOKEN = "Lo da Twilio al crear cuenta"
TWILIO_TWIML_APP_SID = "Lo da Twilio al crear cuenta y una app"
```
Su respuesta es 

```json
{
  "body": "token_de_twilio"
}
```


### /user
La integración que tiene esta ruta es con la lambda `UserService`. La configuracion de esta lambda solo requiere de las siguientes variables de entorno:
```
cluster_arn_aurora = "ARN del cluster"
secret_arn_aurora = "aquí Jose Luis"
```
La peticion a esta ruta es por el método `POST` o `GET` y puede recibir cualquiera de los siguientes JSON  
Definicion de creacion de usuario, ```email``` -> requerido, ```password``` -> requerido
```json
{
    "operation": "create",
    "payload": {
        "Item": {
            "name": "Jose Luis",
            "lastName": "Palillero Huerta",
            "email": "test@digitialconnect.com.mx",
            "phone": "1234567890",
            "rol": "soporte",
            "lastState": "activo",
            "position": "asesor",
            "password": "1234",
            "Dashboard": false,
            "Chat": false,
            "Reportes": false,
            "Respuestas": false,
            "MiCuenta": false,
            "RecuperarPsswrd": false,
            "Consultar": false,
            "Nuevo": false
        }
    }
}
```
Definicion de getItem para los usuarios, ```email``` -> requerido
```json
{
    "operation": "get",
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
            "id": 13,
            "email": "test@digitialconnect.com.mx",
            "lastName": "Palillero Huerta",
            "name": "Jose Luis",
            "phone": "0987654321",
            "rol": "portal",
            "lastState": "activo",
            "position": "asesor",
            "password":"81dc9bdb52d04dc20036dbd8313ed055",
            "Dashboard": false,
            "Chat": false,
            "Respuestas": true,
            "Reportes": false,
            "MiCuenta": false,
            "RecuperarPsswrd": false,
            "Consultar": false,
            "Nuevo": false
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
Para recuperar contraseña
```json
{
    "operation": "recoverPasswd",
    "payload": {
        "Item": {
            "email": "juan.sandoval@digitalconnect.com.mx"
        }
    }
}
```
Para obtener usuarios por rol:
```json
{
    "operation": "getUsersByRol",
    "payload": {
        "Item": {
            "rol": "servicio"
        }
    }
}
```
Para obtener usuarios por nombre y apellido:
```json
{
    "operation": "getByNameLast",
    "payload": {
        "Item": {
            "name": "juan",
            "lastName": "sandoval"
        }
    }
}
```
Para obtener usuarios por estado:
```json
{
    "operation": "getUsersByState",
    "payload": {
        "Item": {
            "lastState": "alta"
        }
    }
}
```
Para obtener todos los usuarios:
```json
{
    "operation": "fullGet",
    "payload": {
        "Item":{}
    }
}
```

### /{folder}/{item}
La integración de esta ruta es con el servicio de `s3` y se encarga de realizar el almacenamiento de un archivo dentro de un bucket ya existente. Su método es `PUT` y el body donde va el archivo debe ser ```application/octet-stream``` un ejemplo de ruta es ```https://17ralen7pg.execute-api.us-west-2.amazonaws.com/dev/datoslex/lex.zip``` donde ```datoslex``` es el nombre del bucket existente y ```lex.zip``` es el nombre con el que se guardar el archivo en el body de la petición

## socket_chatbot
Esta API consta de tres rutas `$connect`, `$disconnect` y `mensaje`. Cada una cuenta con su respectiva lambda con los siguientes nombres `SocketConnect`, `SocketDisconnect` y `SocketMensaje`

### SocketConnect
El dato importante de entrada para esta lambda es el ID de conexión este viene dentro de la siguiente variable ```event['requestContext']['connectionId']```. Este dato debe ser enviado a la base de datos para identificar una nueva conexión y por lo tanto una nueva conversación, así que como configuración de la lambda tiene la siguiente variable de entorno ```ConversationService = ARN de la lambda ConversationService```

### SocketMensaje
Esta lambda es la encargada de procesar los datos de un socket cuando ya está conectado. El JSON que se debe enviar a la ruta `mensaje` es el siguiente

```json
{"action": "mensaje", "datos":"hola"}
```

La lambda recibe los siquientes datos ```event['requestContext']['connectionId']``` es el ID con el que se identifica la conexión ```event['body'])['datos']``` es el mensaje que va hacia la State Machine, la cual esta referenciada en la siguiente variable de entorno ```STATE_MACHINE = "ARN de la state machine chatbot"```. Para invocar la State Machine se utiliza el siguiente código

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
Como los socket no tienen `return` para poder regresar la respuesta de la State Machine se genera el siguiente codigo
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
Esta lambda responde a la ruta `$disconnect`, por el momento no se tiene ninguna funcionalidad solo genera la desconexion segura de un socket (lambda vacía)
