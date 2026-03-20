<!--TRANSLATION_LINKS_START-->
#### Supported by [GitHub Doc Translation](https://github.com/scenery-j/GitHub-Doc-Translation)
> 📖 **Other language versions**：[English (en)](translations/en/README.md)
<!--TRANSLATION_LINKS_END-->

# System Architecture Introduction
### 1. The project provides external real-time communication service (single chat and group chat) while also enabling custom multi-end (web, iOS, Android) message synchronization, offline message caching, online status changes, multi-system access, intelligent dialogue (integrating Tongyi Qianwen large model), etc.
#### 1. Business system: Start Netty service (implement custom protocol) to achieve message sending for single/group chat and other business logic of instant communication.
#### 2. Third-party business system: Implement login and custom business functions for the third-party business system.
#### 3. Frontend service: Implement user interaction for instant messaging.
### 2. Technology Stack
#### Netty: Start Netty server, implement TCP connection, and implement custom protocol to achieve real-time communication between client and server.
#### dubbo: Implement interface calls between TCP microservices and business microservices.
#### rabbitmq: Complete message distribution.
#### mysql: Business data storage.
#### redis: Offline message storage and hot data storage.
#### naocs: Implement service registration and discovery for TCP microservices, and dubbo registry center.
#### Spring Boot: Microservices framework.
### 3. Backend Service Module Introduction
#### im-codec: Implementation related to Netty custom protocol.
#### im-common: Common entity classes and utility classes.
#### im-message-store: Message persistence microservice.
#### im-service: IM service business microservice - implement business logic.
#### im-tcp: Netty service - implement connection with client - message interaction module.
### 4. Business Optimization
#### Message immediacy: That is, how to ensure that the sender's message is timely responded to the receiver. The basic idea is to reduce the single function of the business logic from message sending to message acceptance: 1. Use thread pool to implement concurrency (causing message ordering issues) 2. Message storage uses asynchronous implementation to avoid time-consuming database operations. 3. Move the legality check of message sending forward to the TCP layer, using dubbo remote call to the service layer's business method, to avoid illegal sending requests occupying unnecessary resources.
#### Message ordering: Since message sending uses thread concurrency and asynchronous storage, message ordering cannot be guaranteed. Need to introduce the sequence field, which indicates - implement using Redis increment mechanism to obtain the seq value.
#### Message reliability: To ensure the message is definitely delivered to the receiver: 1. Add an ack indication; when the message reaches the server and reaches the receiver, the client needs to reply with an ack to indicate the sending status. 2. The client provides