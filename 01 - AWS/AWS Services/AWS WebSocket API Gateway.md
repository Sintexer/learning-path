> One implementation of [[AWS API Gateway]]

WebSocket APIs offer APIs that the client can access through the [[WebSocket]] protocol. Unlike [[AWS REST API Gateway|REST]] and [[AWS HTTP API Gateway|HTTP APIs]], WebSocket APIs allow bidirectional communications. WebSocket APIs are often used in real-time applications such as chat applications, collaboration platforms, multiplayer games, and financial trading platforms.

WebSocket APIs maintain a persistent connection between connected clients to facilitate real-time message communication. With WebSocket APIs in API Gateway, you can define backend integrations with Lambda functions, Amazon Kinesis, or any HTTP endpoint to be invoked when messages are received from the connected clients.