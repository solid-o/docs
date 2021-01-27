# Tips 'n tricks

Some tips and recommendations to use Solido at its best.

### 1. Use a framework

It is not mandatory, but it is strongly recommended.  
Frameworks handle security, request parsing and routing for you and also provide fixes and mitigations for security issues.

?> At the moment only the Symfony framework is supported, but you can always help up to provide
an integration with another framework.

### 2. Choose your serializer wisely

We recommend the [Kcs Serializer](https://alekitto.github.io/serializer/) which is a fork of the more popular
[JMS Serializer](https://jmsyst.com/libs/serializer) as it offers some additional features, a simpler way to register
custom serialization handlers and more builtin handlers for common libraries.

We strongly advise against using the Symfony Serializer component, especially when dealing with _enhanced DTOs_
with injected services (via a framework DI component).  
It is however supported, but experience said that it could have some unexpected behavior if not used _really carefully_.

### 3. Use enhanced DTOs

[Versioning](./dto.md?id=versioning) functionalities will save your life (and your workplace).  
You (as every other living human being on Earth) make mistakes, but if the mistakes lands on production, they could be
hard to fix without a proper versioning system which avoid to break _every single client_ that use your APIs.

Write the routes on the interfaces and respect that contract in all the versions (if possible).  
Use `@deprecation` annotation to signal the client that a route is deprecated and will be dismissed soon. Return a 
"Not Found" response for the versions greater than the one which deprecates the route.

### 4. Use URNs to represent relations

When serializing a resource with relations, represents the linked resource(s) with a [URN](./utilities.md?id=uniform-resource-name)
instead of a simple identifier. They are easier to distinguish and can be used in heterogeneous collections as simple strings.

### 5. Do not add foreign data on resource details view...

__Do not__ fall into the temptation of adding related data to a resource when populating and serializing a DTO.  
When responding to a details request (`GET /resources/{id}`), always represent related resources as identifier
(or URN), and not as an object which an `identifer` property.

### 6. ...but add additional data to list entries

When writing a list endpoint ([query language component](./query-language.md?id=query-language) will help you A LOT for this), create
a completely different DTO interface. In its implementation hide properties which are too deep to be useful in a list
and add some _extra_ data, especially on relations.

For example, when writing a list of tasks you can include the name of the task assignee:

```json
/* GET /tasks/123 */
{
    "id": "urn:solido:todo::task:123",
    "title": "Do something beautiful today",
    "description": "A very long description of the task",
    "assignee": "urn:solido:todo::user:42"
}
```

```json
/* GET /tasks */
[
    ...,
    {
        "id": "urn:solido:todo::task:123",
        "title": "Do something beautiful today",
        "assignee": {
            "id": "urn:solido:todo::user:42",
            "name": "John Doe"
        }
    },
    ...
]
```

As you can see, in the list the `description` field has been removed as it is a _very specific_ detail of the task
and the `assignee` field became an object with the `name` property which is likely to be useful when displaying a list.

This will avoid you to request the user details _for every item of the list_ just to display the name of a user.



