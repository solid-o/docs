# CORS

Handles preflight requests and adds CORS headers to API responses.

## Installation

```shell
$ composer require solido/cors
```

## How it works

Public-facing APIs are often queried by web browsers through AJAX requests.  
Modern browsers implements security features including Cross-Origin Resource Sharing (CORS).

?> Please read [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) article
on MDN to know what CORS is and how it works

This library simplify handling of preflight requests and add additional headers to normal responses.

Ex:

```php
$handler = new \Solido\Cors\RequestHandler();
$request = \Symfony\Component\HttpFoundation\Request::createFromGlobals();
if ($request->isMethod('OPTIONS')) {
  $response = $handler->handleCorsRequest($request);  // Generates a response for preflight request
} else {
  $response = ...;
  $handler->enhanceResponse($request, $response);     // Add CORS headers
}

$response->send();
```

## Configuration

This library supports a great variety of cases and can be configured to handle CORS headers differently based
on hostname or path of the request.

In this case an `HandlerFactory` object must be used with the appropriate configuration.

```php
$factory = new \Solido\Cors\HandlerFactory([
  // Global configuration
  'allow_credentials' => true,    // Add Access-Control-Allow-Credentials header
  'allow_origin' => [             // Add Access-Control-Allow-Origin header
    'api.example.org',
    'www.example.org',
    '*',                          // "*" means every origin is accepted
  ],
  'allow_headers' => [            // Add Access-Control-Allow-Headers header
    'Accept',
    'Content-Type',
    'X-Custom-Header',
  ],
  'expose_headers' => [           // Add Access-Control-Expose-Headers to allow javascript to access the specified headers.
    'X-Total-Count'
  ],
  'max_age' => 600,               // Access-Control-Max-Age: how long the preflight request can be cached.
  
  // Local overrides
  paths => [
    [
      'path' => '^/path',         // Host and path in local overrides are regex patterns
      'enabled' => false,
    ],
    [
      'path' => '^/',
      'host' => 'www2\.example\.net',
      'allow_credentials' => false,
    ],
  ]
]);
```

You can then use the configured factory as:

```php
$request = \Symfony\Component\HttpFoundation\Request::createFromGlobals();
$handler = $factory->factory($request->getPathInfo(), $request->getHost());

if ($request->isMethod('OPTIONS')) {
  $response = $handler->handleCorsRequest($request);  // Generates a response for preflight request
} else {
  $response = ...;
  $handler->enhanceResponse($request, $response);     // Add CORS headers
}

$response->send();
```
