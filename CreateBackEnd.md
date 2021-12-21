# ChatbotEngine
Para crear el Pipeline de desarrollo para cualquier chatbot se debe iniciar con los siguientes procesos y comandos
## Creación de Cluster
El bot requiere de una base de datos para crearla es necesario tener instalada la CLI de aws y tener logeado un usuario con permisos de administrador, el comando para hacer la BD es el siguiente
```sh
aws rds create-db-cluster \
--database-name $name$env \
--db-cluster-identifier $name$env \
--engine aurora-mysql \
--engine-version 5.7 \
--master-username $userName \
--master-user-password $userPwd \
--engine-mode serverless \
--scaling-configuration MinCapacity=1,MaxCapacity=1,AutoPause=true,SecondsUntilAutoPause=1000 \
--enable-http-endpoint \
--profile dev
```
Las variables significan: $name nombre del proyecto, $env entorno del proyecto (dev, qna, prod), $userName nombre de usuario para la base de datos, $userPwd contraseña para el usuario de la base de datos.  
## Creación de SecretManager
Para el uso correcto del logeo en la base de datos es necesario utilizar un secretmanage, con el siguiente comando se crea uno.
```sh
aws secretsmanager create-secret \
--name "CredencialDB$name$env" \
--secret-string '{
    "dbInstanceIdentifier": "'$BDName'",
    "engine": "aurora-mysql",
    "host": "'$host'",
    "port": "'$port'",
    "resourceId": "'$clusterID'",
    "username": "'$userName'",
    "password": "'$userPwd'"
}' \
--profile dev
```
Las variables significan: $name nombre del proyecto, $env entorno del proyecto (dev, qna, prod), $BDName nombre de la BD, $host host de la BD, $port puerto de la BD. $clusterID ID de cluster que se encuentra en la consola, $userName nombre de usuario para la base de datos, $userPwd contraseña para el usuario de la base de datos.  
## Creación de ECR
El motor de entrenamiento de requiere de una lambda que se crea con una imagen Docker, por lo tanto se necesita un repositorio, la creacion de este mismo se hace con el siguiente comando
```sh
aws ecr create-repository \
--repository-name "$name$env" \
--image-tag-mutability IMMUTABLE \
--profile dev
```
Las variables significan: $name nombre del proyecto, $env entorno del proyecto (dev, qna, prod).  
## Creación de tablas en BD
El motor del bot, requiere principalmente de dos tablas que se generan con el siguiente comando
```sh
aws rds-data execute-statement \
--resource-arn $Cluster \
--database $BDName \
--secret-arn $SecretsManager \
--sql "CREATE TABLE IF NOT EXISTS Historic (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sessionId VARCHAR(255) NOT NULL,
    mensaje text NULL,
    tipo VARCHAR(255) NOT NULL,
    registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);" \
--profile dev \
aws rds-data execute-statement \
--resource-arn $Cluster \
--database $BDName \
--secret-arn $SecretsManager \
--sql "CREATE TABLE IF NOT EXISTS Conversations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sessionId VARCHAR(255) NULL,
    lastState VARCHAR(255) NULL,
    intent VARCHAR(255) NULL,
    lastQuestion int(11) NULL,
    startConversation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    endConversation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    agent VARCHAR(255) NULL
);" \
--profile dev
```
Las variables significan: $Cluster es el arn del cluster que se puede encontrar en la consola, $BDName nombre de la base de datos, $SecretsManager el arn del secret creado para esa base de datos.
## Creación de S3
Este S3 debe tener permisos para su acceso mediante las lambdas, porque será utilizado por la de entrenamiento, la nomenclatura para su nombre debe ser files-$nameProject-$env  
Falta por redactar  
## Creación de Pipeline
El pipeline requiere de los siguientes puntos:
* Repositorio en github. Este repositorio tendra los archivos necesarios para crear el desarrollo, los archivos indispensables son `buildspec.yaml` y `template.yaml` 
* S3. Este debe tener la siguiente nomenclatura en su nombre pipeline-$nameProject y será el encargado de almacenar todos los despliegues realizados
* Creacion de roles y politicas para Pipeline y CodeBuild. Nomenclatura para los roles y politicas Pipeline$nameProject y Build$nameProject respectivamente. Las politicas necesarias podemos encontrarlas en los archivos `PoliticaPipeline.json` y `PoliticaBuild.json`. En el caso de los roles los encontramos en los archivos `RolBuild.json` y `RolPipeline.json`




El siguiente paso es generar un Pipeline en `CodePipeline`, en donde el origen es `GitHub` en su version 2 y build debe ser `CodeBuild`.  
El proyecto de CodeBuild debe estar en entorno `Linux`, imagen `aws/codebuild/standard:5.0`, Privilegiado debe ser `Verdadero` y CloudWatch Logs `Habilitado`  
Al terminar el proceso de creacion del pipeline, el nombre del S3 creado para el Pipeline se debe agregar en el carchivo `buildspec.yaml` en la linea de `-sam deploy --s3-bucket <pipeline-$nameProject.arn>`  
