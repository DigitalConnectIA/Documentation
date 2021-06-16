# Creación de Aurora serverless

Para la creación de auroraserverless se establece con los siguientes DDL's

```sql

drop table chatbot.Users;
CREATE TABLE IF NOT EXISTS chatbot.Users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    lastName VARCHAR(255) NOT NULL,
    phone VARCHAR(30) NOT NULL,
    password VARCHAR(255) NOT NULL,
    rol VARCHAR(10) NULL,
  	updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

drop table chatbot.Clients;
CREATE TABLE IF NOT EXISTS chatbot.Clients (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    lastName VARCHAR(255) NOT NULL,
    phone VARCHAR(30) NOT NULL,
    updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

drop table chatbot.Conversations;
CREATE TABLE IF NOT EXISTS chatbot.Conversations (
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
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

drop table chatbot.Reports;
CREATE TABLE IF NOT EXISTS chatbot.Reports (
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