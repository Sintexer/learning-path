> AKA **Session layer**

This [[OSI Model]] layer establishes, manages, checkpoints, and terminates communication sessions (dialogs) between communicating **applications**. In the practical TCP/IP stack, session responsibilities are handled by applications or transport/security protocols (e.g., [[TLS]] session resumption, [[HTTP]] cookies/tokens, [[gRPC]]/[[WebSocket]] connection lifecycles).

- **PDU**: Data
- **Examples**: NetBIOS, RPC session controls, [[TLS]] session tickets, [[WebSocket]] sessions.

## Flashcards

OSI Layer 5:::Session layer — establishes, maintains, and terminates logical communication sessions between applications (merged into the Application layer in TCP/IP).

## Related
- [[OSI Model]]
- [[OSI 4 layer]]
- [[OSI 6 layer]]
- [[TLS]]
- [[WebSocket]]
