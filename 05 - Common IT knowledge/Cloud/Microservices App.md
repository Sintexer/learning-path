A microservices app is built from lots of independent small specialised parts that work together to form a meaningful application. For example, you might have an e-commerce app that comprises all of the following small specialised components:
- Web front-end
- Catalog service
- Shopping cart
- Authentication service
- Logging service
- Persistent store
- 
Each of these individual services is called a microservice. Typically, each is coded and owned by a different team. Each can have its own release cycle and can be scaled independently. For example, you can patch and scale the logging microservice without affecting any of the others.

Building applications this way is vital for [[Cloud-Native App]] features.
