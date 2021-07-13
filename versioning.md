# Versioning

Guesses version from request object.

## Installation

```shell
$ composer require solido/versioning
```

## How it works

Versioning is a core feature in solido, the versioning component contains helper classes to detect/guess
version from headers or request attributes.

```php
$guesser = new CustomHeaderVersionGuesser();
$version = $guesser->guess($request, null);
```

### Version attribute on Accept header

Class: `Solido\Versioning\AcceptHeaderVersionGuesser`

Parses `Accept` request header searching for version attribute. For example `Accept: application/json; version=2.3`
will return version `2.3`.  
It accepts an array of mime priorities in case Accept header contains more than one MIME type with different version
attribute (`Accept: text/xml; version=1.0; q=0.4, application/json; version=2.0; q=0.8`).

### Version header on custom header

Class: `Solido\Versioning\CustomHeaderVersionGuesser`

Searches a custom header in the request and uses the value as version. The custom request header should be passed
as parameter to the constructor (defaults to `X-API-Version`).

### Custom guesser

A custom guesser can be used simply implementing `Solido\Versioning\VersionGuesserInterface`.

Example:

```php
class PathPrefixVersionGuesser implements \Solido\Versioning\VersionGuesserInterface {
    public function guess(object $request, ?string $default): ?string {
        $path = $request->getPathInfo();
        if ($match = preg_match('#^/v(\d+\.\d+)/', $path)) {
            return $match[1];
        }

        return $default;
    }
}
```
