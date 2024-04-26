AWS implementation of [[Event-driven architecture]]. Consists of **Event producers**, **Event router**, and **Event consumers**.

**Event producers** create the events. Events contain all of the information required for the consumers to take action on the event. Producers are only aware of the event router. They don't know who the consumer is.

**Event router** ingests, filters and pushes events to the appropriate consumers. It uses a set of rules or another service such as [[AWS SNS]] to send the messages. [[AWS EventBridge]] could be used there.

**Event consumers** either subscribe to receive notification about events or they monitor an event stream and only act on events that pertain to them.

**Event** is a change in state of whatever you are monitoring, for example an updated shopping cart, and entry in a log file, or a new file uploaded to [[AWS S3]].