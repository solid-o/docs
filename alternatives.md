# Alternatives - why not to use ... ?

## FOSRestBundle

It is the first trial to standardize the way to write REST APIs routes and controllers in Symfony.  

It is _controller-based_ (instead of _resource-based_), does not offer _filtering_ or _pagination_ utilities, and contains _too much magic_ in routes registration. 

Otherwise has _decoupled view handling_ and _serialization_ which inspired some of the Solido core concepts.

## API Platform

It is a more complete solution to build a REST API in Symfony. 

It is _resource-centric_ and is great for prototyping and rapid develop (**RAD**) a **CRUD** 
service but it's a bit rigid and is pretty difficult to customize its default behaviors.

It lacks _versioning_ support, limited _list filtering_, _non-RESTful pagination_ support, _not-coherent request
error handling_ and a _limited resource PATCH_ support.  

Additionally it is tightly coupled to [Symfony Serializer](https://symfony.com/doc/current/components/serializer.html) component which has limited functionalities and
is very hard to extend thanks to its _final classes_ paired with very long private methods.

API Platform is _not that modular_: all its components are packaged in one single library and it is nearly impossible to install only the used components.

It also has some good features such as **jsonld** support and **graphql**, but in its implementation there are some major drawbacks working in a domain 
with multiple services, for example referencing their identifiers but not being able to expose their schemas natively. 
There are multiple solutions to workaround this but it's quite a bit of additional effort at the moment of writing.

## Laminas API Tools

Probably the easiest way to create an API project in PHP.

It is obviously _coupled to the Laminas project_ (formerly **Zend Framework**). 

It is _feature complete_, but the project went probably _out of scope_ handling 
non-API related things like authentication via OAuth or social log in.
