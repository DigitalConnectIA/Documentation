# Creación de tablas en Aurora serverless

Para la creación de las tablas en auroraserverless se establece con los siguientes DDL's

```sql
CREATE TABLE IF NOT EXISTS Agents ( 
    id int(11) NOT NULL AUTO_INCREMENT,
    email varchar(255) NOT NULL,
    name varchar(255) NOT NULL,
    lastName varchar(255) NOT NULL,
    phone varchar(30) NOT NULL,
    password varchar(255) NOT NULL,
    rol varchar(10) DEFAULT NULL,
    lastState varchar(255) DEFAULT NULL,
    position varchar(255) DEFAULT NULL,
    updated timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    created timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    Dashboard tinyint(1) DEFAULT NULL,
    Chat tinyint(1) DEFAULT NULL,
    Reportes tinyint(1) DEFAULT NULL,
    Respuestas tinyint(1) DEFAULT NULL,
    MiCuenta tinyint(1) DEFAULT NULL,
    RecuperarPsswrd tinyint(1) DEFAULT NULL,
    Consultar tinyint(1) DEFAULT NULL,
    Nuevo tinyint(1) DEFAULT NULL,
    sessionId varchar(255) DEFAULT NULL,
    sessionUser varchar(255) DEFAULT NULL, PRIMARY KEY (id)
);

CREATE TABLE IF NOT EXISTS Historic (
    id int(11) NOT NULL AUTO_INCREMENT,
    sessionId varchar(255) NOT NULL,
    mensaje text,
    tipo varchar(255) NOT NULL,
    registro timestamp NULL DEFAULT CURRENT_TIMESTAMP, PRIMARY KEY (id)
);

CREATE TABLE IF NOT EXISTS Reports (
    id int(11) NOT NULL AUTO_INCREMENT,
    reportId varchar(255) NOT NULL,
    agent varchar(255) NOT NULL,
    comment text NOT NULL,
    priorityAttention tinyint(1) DEFAULT NULL,
    processStatus varchar(255) NOT NULL,
    updated timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    created timestamp NULL DEFAULT CURRENT_TIMESTAMP, PRIMARY KEY (id)
);

CREATE TABLE IF NOT EXISTS Users (
    id int(11) NOT NULL AUTO_INCREMENT,
    sessionId varchar(255) DEFAULT NULL,
    lastState varchar(255) DEFAULT NULL,
    intent varchar(255) DEFAULT NULL,
    lastQuestion int(11) DEFAULT NULL,
    startConversation timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    endConversation timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    onHold tinyint(1) DEFAULT NULL, PRIMARY KEY (id)
);

```