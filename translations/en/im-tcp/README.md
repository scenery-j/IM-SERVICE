# ModuleImplementation Overview
## Netty, WebSocket Server  Use Spring's startup event; when the ContextRefreshedEvent container finishes starting up, start the Netty and WebSocket servers; when the ContextClosedEvent container shuts down, close the clients

## Custom Protocol Implementation