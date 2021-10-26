# Creación de tablas en Aurora serverless

Para la creación de las tablas en auroraserverless se establece con los siguientes DDL's

```sql

drop table DBName.Users;
CREATE TABLE IF NOT EXISTS DBName.Users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    lastName VARCHAR(255) NOT NULL,
    phone VARCHAR(30) NOT NULL,
    password VARCHAR(255) NOT NULL,
    rol VARCHAR(10) NULL,
    lastState VARCHAR(255) NULL,
    position VARCHAR(255) NULL,
  	updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    Dashboard boolean NULL,
    Chat boolean NULL,
    Reportes boolean NULL,
    Respuestas boolean NULL,
    MiCuenta boolean NULL,
    RecuperarPsswrd boolean NULL,
    Consultar boolean NULL,
    Nuevo boolean NULL,
    sessionId VARCHAR(255) NULL
);

drop table DBName.Clients;
CREATE TABLE IF NOT EXISTS DBName.Clients (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    lastName VARCHAR(255) NOT NULL,
    phone VARCHAR(30) NOT NULL,
    updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

drop table DBName.Conversations;
CREATE TABLE IF NOT EXISTS DBName.Conversations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sessionId VARCHAR(255) NULL,
    userMessage text NULL,
    lastState VARCHAR(255) NULL,
    intent VARCHAR(255) NULL,
    lastQuestion int(11) NULL,
    botMessage text NULL,
    startConversation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    endConversation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    agent VARCHAR(255) NULL
);

drop table DBName.Reports;
CREATE TABLE IF NOT EXISTS DBName.Reports (
    id INT AUTO_INCREMENT PRIMARY KEY,
    reportId VARCHAR(255) NOT NULL,
    agent VARCHAR(255) NOT NULL,
    comment text NOT NULL,
    priorityAttention boolean NULL,
    processStatus VARCHAR(255) NOT NULL,
    updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```  

Diagrama Entidad-Relacion  

![DiagramaER](https://referencias-documentacion-md.s3.us-west-2.amazonaws.com/DiagramaER.jpg)  
