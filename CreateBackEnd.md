# ChatbotEngine
Para la implementacion de la plantilla es necesario cambiar variables globales en el siguiente archivo `buildspec.yaml`  
En este archivo se debe cambiar, segun a consideracion, las variables:  
* Env: es el sufijo que se le dará a todas las aplicaciones desplegadas, esto para diferenciar de las otras plantillas
* DBUsername: Nombre de usuario con el cual se identificara en la base de datos de aurora
* DBPassword: Contraseña para el usuario de la base de datos de Aurora
* DBName: Nombre de la base de datos Aurora  
* BotName: Nombre del Lex el cual `SFCualquierMensaje` invoca
* BotAlias: Alias del mismo bot de Lex, se crea despues de implementar el bot  

Se deben crear las siguientes politicas
* PipelineChatbot: Los permisos que debe llevar se encuentran en el json `PipelineChatbotPolitica.json`
* BuildChatbot: Los permisos que debe llevar se encuentran en el json `BuildChatbotPolitica.json`, algunos campos dentro del json se deben referenciar al numero de cuenta en el que se va implementar  

Una vez generadas las politicas, se deben crear los siguientes roles y asignar sus respectivas politica segun el nombre que les corresponda
* PipelineChatbot: En las relaciones de confianza debe llevar el json `RelacionConfianzaPipelineRol.json` y en la politica `PipelineChatbot`
* BuildChatbot: En las relaciones de confianza debe llevar el json `RelacionConfianzaBuildRol.json` y en la politica `BuildChatbot`  

Posteriormente mediante la CLI AWS de manera local, se debe desplegar un repositorio donde se almacenara la imagen de la lambda `SnipsTrainer`, por lo tanto, se debe ejecutar el siguiente comando  

```
aws ecr create-repository --repository-name <NombreDelProyecto> --image-tag-mutability IMMUTABLE
```

Debe regresar un json y se tiene que copiar el URI y pegar en `buildspec.yaml` en la linea de `-sam deploy --image-repository <nuevo repositorio>`. Posteriormente se tiene que ir a la consola de ECR para asignar al repositorio los permisos del json `politicaECR.json`  

El siguiente paso es generar un Pipeline en `CodePipeline`, en donde el origen es `GitHub` en su version 2 y build debe ser `CodeBuild`.  
El proyecto de CodeBuild debe estar en entorno `Linux`, imagen `aws/codebuild/standard:5.0`, Privilegiado debe ser `Verdadero` y CloudWatch Logs `Habilitado`  
Al terminar el proceso de creacion del pipeline, creara un Bucket S3 se debe copiar el nombre y agregar en el carchivo `buildspec.yaml` en la linea de `-sam deploy --s3-bucket <nuevo bucket>`  
