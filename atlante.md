# Atlante

Atlante is a set of client libraries for various languages.  
All the clients have similar functionalities, behaviors and names (where possible) and implement language standards if present.

> Curiosity: _Atlante_ is the italian name of the _Atlas_ titan which, according to Greek mythology, holds
> up all the celestial heavens and thus the entire world on his shoulders.

## Installation

Javascript:

```shell
$ npm install @solido/atlante-js
```

PHP:

```shell
$ composer require solido/atlante-php
```

## Client

The main client class (usually called `Client`) should be initialized passing a requester object (which effectively
executes HTTP requests) and a set of [request decorators](./atlante.md?id=request-decorators).

It exposes one main method `request` which allows you to perform a generic HTTP request and some shortcuts for
the more common http methods (`get`, `post`, `put`, `patch` and `delete`).

## Requesters

A requester is an object which performs an HTTP request through libraries or system facilities (for example through
`XMLHttpRequest` object or a PSR-7 compliant http client for PHP).

The request data will be passed already transformed and/or decorated.

## Request decorators

Request decorators enhance client requests for example applying a prefix to the URL, or adding authorization headers.

Bundled request decorators:

| Class                        | Available in | Description 
| ---------------------------- | ------------ | -------------
| `AcceptFallbackDecorator`    | PHP          | Add Accept header if not present
| `BodyConverterDecorator`     | JS, PHP      | Converts body in various formats (array, object, function, iterator, etc) to string
| `ClientTokenAuthenticator`   | JS, PHP      | Add OAuth client token Bearer authorization header (exchanging client id and secret)
| `CodeFlowAuthenticator`      | JS           | Add OpenID connect authorization token (requested with code flow)
| `HttpBasicAuthenticator`     | JS, PHP      | Add HTTP basic authorization header
| `ImplicitFlowAuthenticator`  | JS           | Add OpenID connect authorization token (requested with implicit flow)
| `TokenPasswordAuthenticator` | JS           | Add OAuth Bearer authorization token (requested with password grant)
| `UrlDecorator`               | JS, PHP      | Add an URL prefix
| `VersionSetterDecorator`     | JS, PHP      | Add version parameter to Accept header

## Failing requests

Non-2xx responses makes client to throw an exception.  
`Bad request`, `Not found` and `Forbidden` status codes have its own exception objects, which obviously
share the same parent class.
