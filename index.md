# Solido
__A rock solid toolbox for building REST API in PHP__

## Introduction

Solido is a set of libraries to build rock solid, maintainable and reliable REST APIs in PHP.

It provides tools for CORS handling, versioning, security, request parsing, list pagination, filtering and serialization.

It also provides integration with the popular PHP framework Symfony (integrations with other frameworks are welcome too!)

## Philosophy

Solido is built as a set of libraries and tools that eases the development of a REST API in a framework agnostic way.  
Framework integrations are welcome (at the moment only the Symfony one is present) but are not needed to run the standalone components.

If you are interested why we decided not to use other php alternative libraries take a look to our [FAQ](./faq.md) section.

## Components

- [body-converter](./body-converter.md?id=body-converter): normalize body into a request object to be handled without special Content-Type checks
- [cors](./cors.md?id=cors): handles CORS headers and OPTIONS requests
- [data-transformers](./data-transformers.md?id=data-transformer): set of text to object transformers, used to convert request data
- [dto-management](./dto-management.md?id=dto-management): DTOs are first class citizens in Solido. Versioning, patching, serialization, 
  even routing happens on DTOs. For more information see the [DTO section](./dto.md?id=data-transfer-objects)
- [pagination](./pagination.md?id=pagination): handles pagination (and endless-pagination) in RESTful ways.
- [patch-manager](./patch-manager.md?id=patch-manager): handles the PATCH request. Can understand 
  [marge-patch](https://tools.ietf.org/html/rfc7386) requests as well as json patch described
  in [RFC 6902](https://tools.ietf.org/html/rfc6902) and handles them in one single operation.
- [query-language](./query-language.md?id=query-language): powerful query language for REST lists, can automatically build doctrine
  queries and can be extended to support custom filtering
- security-policy-checker: utilities to check complex security policies based on route, resource and other conditions [WIP]
- [serialization](./serialization-component.md?id=serialization): provides abstraction to serializers.
- [versioning](./versioning.md?id=versioning): analyses the request and guesses the version request by the user.

## Framework integrations

- [symfony](./symfony-integration.md?id=symfony-integration): provides integration with Symfony framework and security component. Enables per-field security on DTOs.

## Client libraries

API consumers are also taken into consideration: Atlante project and libraries has been developed to help api consumers with useful classes and utilities.

See [Atlante](./atlante.md) for further information.

## License

All the solido libraries and components are releases under the business-friendly MIT license.  
The documentation is released under CC0 license.

## Contributing

Contributions, issues and PRs are welcome. See the [contributing](./CONTRIBUTING.md) section for information.
