# Utilities

## Uniform Resource Name

Uniform Resource Names (URN) can be used as universal identifiers of your resources to unequivocally identify a resource
from a string which includes not only a simple identifier, but also its class, owner and application domain.

Solido includes some utility classes which help you to generate and serialize a URN and to retrieve an entity
from its URN in the `solido/common` library.

### Urn

The solido implementation of URN expects the resource identifier, its class, an optional owner id, an optional 
tenant id (for multi-tenant applications or sharded databases), an optional partition id (ex: the project name),
and a domain (ex: the company name).

If empty, the domain will be set as `Urn::$defaultDomain` static property. This property is public and can be
set when the application starts to a fixed value.

?> Symfony integration will set the default domain on bundle `boot` method.<br>
The default domain can be set in the [bundle configuration](./symfony-integration.md?id=configuration)

The urn implements the `Stringable` interface; its string representation is:

```
urn:domain:partition:tenant:owner:class:identifier
```

!> The `urn:` protocol is fixed and cannot be changed.

For example, a valid urn can be

```
urn:my-company:todo::1047:task:24
```

It unequivocally represents the `Task` with id `24` owned by user `1047` of the `ToDo` project
developed by `My company`. It this case the tenant is unused, so it is left empty.

Another valid urn is `urn:my-company::::task:74`. In this example partition, tenant and owner are omitted.

### Urn generator

Every entity capable of generating its own urn can implement the `UrnGeneratorInterface`.  
The `UrnGeneratorTrait` can be used with the interface to include the default implementation which
omits the partition, the tenant and the owner ids, produces a slugged version of the class and needs only
the `getUrnId` to be implemented.

Example:

```php
src/Entity/Task.php

class Task implements UrnGeneratorInterface {
    use UrnGeneratorTrait;
    
    private int $id;
    private string $description;

    // ...

    public function getUrnId(): string {
        return (string) $this->id;
    }
}
```

### Urn converter

Entity/document can be retrieved from its urn representation thanks to the `UrnConverter` class.  
It currently works with doctrine `ManagerRegistry` objects and searches for the correct entity from its properties.

Its `getItemFromUrn` accepts an Urn object, and an optional string representing the acceptable class to be converted.
If no object is found or the object found is not an instance of the acceptable class, a 
`Solido\Common\Exception\ResourceNotFoundException` exception is thrown.

```php
$em = getEntityManager();
$cacheDir = __DIR__ . '/cache_urn';

$converter = new \Solido\Common\Urn\UrnConverter([ $em ], new \Symfony\Component\Config\ConfigCacheFactory(false), $cacheDir);
$object = $converter->getItemFromUrn(new Urn('urn:my-company::::task:74'), Task::class);

assert($object instanceof Task && 74 === $object->getId());
```
