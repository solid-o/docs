# Alternatives - why not to use ... ?

## FOSRestBundle

It is the first trial to standardize the way to write REST APIs routes and controllers in Symfony.  
It is limited in its features as do not offer filtering or pagination utilities, is controller-based 
(and not resource-based) and contains too much magic in routes registration, but has decoupled
view handling and serialization which inspired some concepts of Solido.

## API Platform

It is a more complete solution to build a REST API in Symfony. It is resource-centric but has
no versioning support at all, limited list filtering, non-RESTful pagination support, non-coherent request
error handling, limited resource PATCH support and an overall orientation to CRUD and RAD which makes very
difficult to customize its behaviors.  
Additionally it is tightly coupled to Symfony Serializer component which has limited functionalities and
very hard to extend thanks to its final classes paired with very long private methods.

All its components are packaged in one single library and it is nearly impossible to install only the used components.

It has some good features such as jsonld support and graphql, but they are major drawbacks when working
with multiple applications or microservices as the application do not know anything about the other services’ resources.

However it is great for prototyping and rapid development, but cannot be really used for a serious
project with external integration.

## Laminas API Tools

Probably the easiest way to create an API project in PHP, it is obviously coupled to the Laminas project
(formerly Zend Framework). It is feature complete, but the project went probably out of scope handling 
non-API related things like authentication via OAuth or social log in.
