Identity federation allows users with passwords elsewhere - like corporate network or internet identity provider - to get access to a resource. E.g. allow users from LDAP to access application via [[Keycloak]]. 

Identity federation is a system of trust between two parties for the purpose of authenticating users and conveying information needed to authorize their access to resources. In this system, an [[Identity provider]] is responsible for user authentication, and a service provider, such as a service or an application, controls access to resources.

The flow:

1. A trust relationship is configured between the identity provider and the service provider. The service provider trusts the identity provider to [[Authentication and Authorization#Authentication|authenticate users]] and relies on the information provided by the identity provider about the users.
2. After authenticating a user, the identity provider returns a message, called an assertion, containing the user's sign-in name and other attributes that the service provider needs to establish a session with the user and to determine the scope of resource access.
3. The service provider receives the assertion from the user, validates the level of access requested, and sends the user the necessary credentials to access the desired resource.
4. With the right credentials from the service provider, the user has now direct access to the requested resources via an established session.