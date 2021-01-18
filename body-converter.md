# Body converter

Decodes and normalizes request body into a manageable parameter bag.

## Installation

```shell
$ composer require solido/body-converter
```

## How it works

APIs can accept request in various formats (JSON, form-data, url-encoded, etc).  
This library reads the request body (and headers) and normalizes it into a `ParameterBag` object.

```php
$request = \Symfony\Component\HttpFoundation\Request::createFromGlobals();

$bodyConverter = new \Solido\BodyConverter\BodyConverter();
$parameters = $bodyConverter->decode($request);
```

This is done automatically by framework integrations to avoid special `Content-Type` checks
throughout the request lifecycle.

## Supported formats

Supported `Content-Type` includes:

| MIME                              | Format |
| --------------------------------- | ------ |
| application/x-www-form-urlencoded | form   |
| application/json                  | json   |
| application/merge-patch+json      | json   |

If a not decodable format is encountered, the `decode` method will return an empty parameter bag.

?> The default implementation does not throw an exception in case of unknown format in order to allow
 file upload as request body. If you want to discuss this behavior, please file an
 issue on [body-converter repository](https://github.com/solid-o/body-converter)
