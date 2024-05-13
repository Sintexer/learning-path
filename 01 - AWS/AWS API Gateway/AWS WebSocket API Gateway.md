> One implementation of [[AWS API Gateway]]

WebSocket APIs offer APIs that the client can access through the [[WebSocket]] protocol. Unlike [[AWS REST API Gateway|REST]] and [[AWS HTTP API Gateway|HTTP APIs]], WebSocket APIs allow bidirectional communications. WebSocket APIs are often used in real-time applications such as chat applications, collaboration platforms, multiplayer games, and financial trading platforms.

WebSocket APIs maintain a persistent connection between connected clients to facilitate real-time message communication. With WebSocket APIs in API Gateway, you can define backend integrations with Lambda functions, Amazon Kinesis, or any HTTP endpoint to be invoked when messages are received from the connected clients.

You might want to only allow certain clients to call your API, or you might want it to be available to everyone. In addition, you might want an API call to invoke a Lambda function, make a database query, or call an application. All of these options will change the characteristics of the API as you design and deploy it.

## WebSocket connection management

To understand how the WebSocket connections are maintained, you need to understand how the client connects, sends messages, and disconnects from the API.

**Connect**: The client apps connect to your WebSocket API by sending a WebSocket upgrade request. If the request succeeds, the `$connect` route is invoked while the connection is being established. Until the invocation of the integration you associated with the `$connect` route is completed, the upgrade request is pending and the actual connection will not be established. If the `$connect` request fails, the connection will not be made.

**Message**: After the connection is established, your client's JSON messages can be routed to invoke a specific backend service based on message content. When a client sends a message over its WebSocket connection, this results in a route request to the WebSocket API. The request will be matched to the **route** with the corresponding route key in API Gateway.

**Disconnect**: The `$disconnect` route is invoked after the connection is closed. The connection can be closed by the server or by the client. Since the connection is already closed when it is invoked, the `$disconnect` route is a best-effort event. API Gateway will try its best to deliver the `$disconnect` event to your integration, but it cannot guarantee delivery. The backend can initiate disconnection by using the `@connections` API.

## Route

With WebSocket APIs in API Gateway, JSON messages can be routed to invoke a specific backend service based on message content. When a client sends a message over its WebSocket connection, this results in a route request to the WebSocket API. The request will be matched to the route with the corresponding route key in API Gateway. 

There are three predefined routes that can be used with WebSocket APIs: `$connect`, `$disconnect`, and `$default`. In addition to the predefined routes, you can also create custom routes. 

Possible route types:

1. `$connect` - API Gateway calls the `$connect` route when a persistent connection between the client and a WebSocket API is being initiated.
2. `$disconnect` - API Gateway calls the `$disconnect` route when the client or sever disconnects from the API. The `$disconnect` route is invoked after the connection is closed. The connection can be closed by the server or the client. As the connection is already closed when it is executed, `$disconnect` is a best-effort event. API Gateway will try its best to deliver the `$disconnect` event to your integration, but **it cannot guarantee delivery**.
3. `$default` - Every API Gateway WebSocket API can have a default route. This is a special routing value that can be used as a fallback route, route for Non-JSON payload, or without any route keys to just proxy request to a backend component.
4. **Custom route** - to invoke a specific backend component (**integration**) based on the message content.

## Integration

After setting up an API route, you must integrate it with an endpoint in the backend. A backend endpoint is also referred to as an **integration endpoint** and can be a [[AWS Lambda function]], an HTTP endpoint, or an AWS service action. The API integration has an integration request and an integration response option.

AWS WebSocket API could be configured for one-way or two-way communication. One-way communication will pass request to the integration and won't send any request back. WebSocket API might be one-way. But if you set-up flow for response handling, your WebSocket API becomes two-way. One-way communication routes do not return a response over the WebSocket channel. Two-way communication routes return a response after processing the message.

Integration requests and responses allow for data transformation and customization of communication.

For **one-way** communication, an integration request is attached to the route. This involves:

- **Choosing a route key:** This can be a predefined key like `$connect`, `$disconnect`, `$default`, or a custom key.
- **Specifying the backend endpoint:** This is the AWS service or HTTP endpoint to invoke for the chosen route.
- **Configuring request templates:** These transform the route request data into integration request data.

For **two-way** communication, a route response is configured. This allows for transformations on the returned message payload, similar to [[AWS REST API Gateway|REST API]] integration responses.

## Selection expressions

Selection expressions are powerful tools used in AWS WebSocket API Gateway to **dynamically** choose the appropriate response, API key requirement, or API stage based on the *context of the request*. These expressions evaluate various factors, such as the request path, headers, or query parameters, and use the resulting value to select from a predefined set of options.

By leveraging selection expressions, you can create highly adaptable and flexible WebSocket APIs that cater to diverse scenarios and client needs. These expressions enable you to:

- **Route Response Selection Expressions (Tailor responses)**: Send specific responses to clients based on their requests, enhancing the overall user experience. These expressions help you choose the right response to send back to the client based on the context of the request.
- **API Key Selection Expressions (Enforce security)**: Require API keys only when necessary, adding an extra layer of protection for sensitive data. These expressions determine whether a client needs to provide a valid API key to access the WebSocket API.
- **API Mapping Selection Expressions (Route requests efficiently)**: Direct incoming requests to the appropriate API stage, ensuring optimal performance and scalability. These expressions decide which API stage to use when a request comes through a custom domain.

## Pricing

Pricing options are set per [[AWS API Gateway stage|stage]].

**Pay for message count**: WebSocket APIs for API Gateway charge for the messages you send and receive. You can send and receive messages up to 128 KB in size. Messages are metered in 32-KB increments, so a 33-KB message is charged as two messages. For WebSocket APIs, the API Gateway free tier currently includes one million messages (sent or received) and 750,000 **connection minutes** for up to 12 months.

**Connection minutes**: In addition to paying for the messages you send and receive, you are also charged for the total number of connection minutes.

**Additional charges**: You may also incur additional charges if you use API Gateway in conjunction with other AWS services or transfer data out of AWS. 