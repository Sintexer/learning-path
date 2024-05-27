**GraphQL** is a data query and manipulation language for APIs, and a runtime for executing those queries with your existing data. It was developed by Facebook in 2012 and later open-sourced.

**Efficiency** is one of the key advantages of GraphQL. Unlike traditional [[REST|REST APIs]], with GraphQL, the client specifies exactly what data it needs, which can reduce the amount of data that needs to be transferred over the network. This makes applications using GraphQL more efficient and faster.

**Single Request**. GraphQL allows the client to make a single request to fetch the required data rather than making several requests to fetch the same data. This can greatly reduce the number of round trips between the client and server.

**Strong Typing** is another important aspect of GraphQL. Every GraphQL schema is strongly typed. This means each data point has a specific type, and all types are defined in the schema. This leads to clearer API contracts and better tooling.

**Real-Time Updates** with GraphQL are implemented with subscriptions. Clients subscribe to updates they're interested in, and the server pushes updates to those clients as they happen.

**Integration** with existing code is straightforward with GraphQL. It can wrap existing REST, [[gRPC]], or other API endpoints, or work directly with your database, providing a unified interface for all your data.