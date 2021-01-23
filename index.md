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

- [body-converter](./body-converter.md): normalize body into a request object to be handled without special Content-Type checks
- [cors](./cors.md): handles CORS headers and OPTIONS requests
- [data-transformers](./data-transformers.md): set of text to object transformers, used to convert request data
- [dto-management](./dto-management.md): DTOs are first class citizens in Solido. Versioning, patching, serialization, 
  even routing happens on DTOs. For more information see the [DTO section](./dto.md)
- [pagination](./pagination.md): handles pagination (and endless-pagination) in RESTful ways.
- [patch-manager](./patch-manager.md): handles the PATCH request. Can understand 
  [marge-patch](https://tools.ietf.org/html/rfc7386) requests as well as json patch described
  in [RFC 6902](https://tools.ietf.org/html/rfc6902) and handles them in one single operation.
- [query-language](./query-language.md): powerful query language for REST lists, can automatically build doctrine
  queries and can be extended to support custom filtering
- security-policy-checker: utilities to check complex security policies based on route, resource and other conditions [WIP]
- serialization: provides abstraction to serializers. See [Serialization section](./serialization.md) for more information
- [versioning](./versioning.md): analyses the request and guesses the version request by the user.

## Framework integrations

- symfony: provides integration with Symfony framework and security component. Enables per-field security on DTOs.

## Client libraries

API consumers are also taken into consideration: Atlante project and libraries has been developed to help api consumers with useful classes and utilities.

See [Atlante](./atlante.md) for further information.

## License

All the solido libraries and components are releases under the business-friendly MIT license.

## Contributing

Contributions, issues and PRs are welcome. See the [contributing](./contributing.md) section for information.
