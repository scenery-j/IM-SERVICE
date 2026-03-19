# System Architecture Overview
### I. This project provides instant messaging services (one-on-one and group chats) while enabling custom multi-platform (web, iOS, Android) message synchronization, offline message caching, online status changes, multi-system integration, and intelligent dialogue (integrated with Tongyi Qianwen model).
#### 1. Business System: Starts Netty service (implements custom protocol) to handle message sending for one-on-one/group chats and other instant messaging business logic.
#### 2. Third-party Business System: Implements login and custom business functions for third-party systems.
#### 3. Frontend Service: Handles user interaction for instant messaging.

### II. Technology Stack
#### Netty: Starts Netty server, establishes TCP connections, and implements custom protocol for real-time communication between client and server.
#### Dubbo: Facilitates interface calls between TCP microservices and business microservices.
#### RabbitMQ: Handles message distribution.
#### MySQL: Stores business data.
#### Redis: Stores offline messages and hot data.
#### Nacos: Implements service registration and discovery for TCP microservices and Dubbo registry.
#### Spring Boot: Microservice framework.

### III. Backend Service Modules
#### im-codec: Implements Netty custom protocol.
#### im-common: Contains common entity classes and utility classes.
#### im-message-store: Message persistence microservice.
#### im-service: IM business microservice - implements business logic.
#### im-tcp: Netty service - handles client connections and message interaction module.

### IV. Business Optimization
#### Message Timeliness: Ensures messages from the sender are promptly delivered to the receiver by reducing the complexity of the message delivery process: 1. Uses thread pools for concurrency (may cause message ordering issues). 2. Implements asynchronous message storage to avoid time-consuming database operations. 3. Validates message sending legality at the TCP layer using Dubbo remote calls to service layer methods, preventing unnecessary resource consumption by invalid requests.
#### Message Ordering: Since message sending uses concurrency and asynchronous storage, message ordering cannot be guaranteed. Introduces a sequence field - uses Redis auto-increment mechanism to obtain seq values.
#### Message Reliability: Ensures messages are delivered to the receiver: 1. Adds ACK mechanism, where the server and receiver must reply with ACK to indicate message delivery status. 2. Provides client-side fault tolerance mechanism, requiring retransmission if ACK is not received (causes idempotency issues).
#### Message Idempotency: Since messages may be retransmitted, the client should only receive the message once. Ensures the same message is processed only once by using Redis to store processed message keys - sets timeout, skips processing if the key exists in cache.

![System Architecture Overview](https://github.com/user-attachments/assets/37f8f29a-a389-4578-86f8-c61b3915f432)
# Data Flow Sequence Diagram
![Data Flow Sequence Diagram](https://github.com/user-attachments/assets/ed849f9d-4797-40e5-9248-a5b2fe298a1f)

# Features
### 1. Login Page
![Login Page](https://github.com/user-attachments/assets/b87d0d1c-2809-415f-9f7d-4a35a6c29025)
### 2. Chat Homepage
![Chat Page](https://github.com/user-attachments/assets/e5325ee7-3309-4bd1-ab1c-44e3048bc69f)
### 3. Intelligent Q&A
![Intelligent Q&A](https://github.com/user-attachments/assets/61fda008-da21-44fd-b8be-7502724a50c9)
### 4. Add Friend Page
![image](https://github.com/user-attachments/assets/a21ba3c0-d3a9-4013-a460-a1fa9d881bbf)
### 5. Friend Details Page
![image](https://github.com/user-attachments/assets/9c0ae696-ea16-4d91-9eaa-cd53879f0175)
### 6. One-on-One Chat Display
![One-on-One Chat Display](https://github.com/user-attachments/assets/1b9ef42c-b37b-49ba-91bc-3a2d9a6e550d)
### 7. Group Chat Display
![Group Chat Display](https://github.com/user-attachments/assets/25e011db-b2ca-4bc5-b444-c7e1d81afbf6)
### 8. Logout Page
![Logout Page](https://github.com/user-attachments/assets/aef6d0c7-ad26-402a-a56d-31a4fb051cc4)



# IM-SERVICE Instant Messaging System

A distributed instant messaging system based on Spring Cloud and Netty.

## System Architecture

The system includes the following main services:

- `im-service`: Core business service
- `im-message-store`: Message storage service
- `im-tcp`: TCP long connection service

## Environment Requirements

- JDK 8+
- Maven 3.6+
- Docker 20.10+
- Docker Compose 2.0+

## Quick Start

### 1. Project Packaging

%%CODEBLOCK_0%%

### 2. Build Microservice Images

#### 2.1 Build Base Image

First, build a Docker image containing the base environment, which all services will be based on:

%%CODEBLOCK_1%%

#### 2.2 Build Service Images

Use the following commands to build images for each service:

%%CODEBLOCK_2%%

Or use docker-compose to build all service images at once:

%%CODEBLOCK_3%%

#### 2.3 Image Build Instructions

The Dockerfile for each service mainly includes the following steps:

1. im-service Dockerfile:
%%CODEBLOCK_4%%

2. im-message-store Dockerfile:
%%CODEBLOCK_5%%

3. im-tcp Dockerfile:
%%CODEBLOCK_6%%

#### 2.4 Image Management Commands

%%CODEBLOCK_7%%

### 3. Build Infrastructure

The project depends on the following basic services:

- MySQL 8.0: Data persistence
- Redis 6.2: Cache service
- RabbitMQ 3.x: Message queue
- Nacos 2.2.3: Service registration and discovery

These services are already configured in docker-compose.yml and do not require separate installation.

### 4. Initialize Configuration

%%CODEBLOCK_8%%

### 5. Start Services

%%CODEBLOCK_9%%

Service startup sequence:
1. Infrastructure services (MySQL, Redis, RabbitMQ, Nacos)
2. im-service
3. im-message-store
4. im-tcp
![image](https://github.com/user-attachments/assets/b40a4ec0-25ee-496a-a851-a93175644156)


### 6. Service Access

After starting the services, they can be accessed via the following addresses:

- Nacos Console: http://localhost:8848/nacos
  - Username: nacos
  - Password: nacos
- RabbitMQ Management Interface: http://localhost:15672
  - Username: guest
  - Password: guest
- Business Service Interfaces:
  - im-service: http://localhost:8000
  - im-message-store: http://localhost:8001
  - im-tcp: 
    - HTTP: http://localhost:8002
    - TCP: localhost:9000

### 7. Common Maintenance Commands

%%CODEBLOCK_10%%

### 8. Data Persistence

All service data is persisted via Docker volumes:

- MySQL data: mysql_data
- Redis data: redis_data
- RabbitMQ data: rabbitmq_data
- Nacos data: nacos_data

### 9. Network Configuration

All services run under the `im-network` network, and services access each other via service names:

- MySQL: im-mysql:3306
- Redis: im-redis:6379
- RabbitMQ: im-rabbitmq:5672
- Nacos: im-nacos:8848

### 10. Troubleshooting

1. Service startup failure
   %%CODEBLOCK_11%%

2. Network connection issues
   %%CODEBLOCK_12%%

3. Data persistence issues
   %%CODEBLOCK_13%%

### 11. Notes

1. Ensure Maven is correctly installed and configured during the first startup
2. Ensure Docker and Docker Compose are correctly installed
3. Ensure required ports are not occupied
4. Service startup may take some time, please be patient
5. If encountering permission issues, run commands with administrator privileges

## Technology Stack

- Spring Boot
- Spring Cloud
- Netty
- MySQL
- Redis
- RabbitMQ
- Nacos
- Docker
- Docker Compose

## Contribution Guide

Welcome to submit Issues and Pull Requests.

## License

[MIT License](LICENSE)