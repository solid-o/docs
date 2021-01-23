# Data Transfer Objects

DTOs are first class citizens in solido being the contact point of the request, the view and the database model.

They also have a central role in versioning and can be enhanced (thanks to the [dto management component](./dto-management.md))
with extensions which generate code that augments the functionalities of the DTO (and enables magic!).

## A simple DTO

You can use simple DTOs just to prepare data for serialization.  
The DTO should be populated with entity data and then serialized into JSON or XML.

An example of simple DTO could be:

```php
class MyDTO {
    public $name;
    public $surname;
    public $email;
    
    public function get(User $entity): void
    {
        $this->name = $entity->getName();
        $this->surname = $entity->getSurname();
        $this->email = $entity->getEmail() ?? '<Email not available>';
    }
}
```

Then you can use it:

```php
$user = $db->find(User::class, $id);
$dto = new MyDTO();
$dto->get($user);

echo $serializer->serialize($dto, 'json');
```

## Versioning

In a real world application you need a powerful versioning system.  
And that's because people (and developers) make mistakes or business does a *little small tiny* change which
makes your beautiful application logic collapse like a house of cards.

Solido was born with a strong versioning engine in mind, and a central part of it are the DTOs.

All the versions of the same DTO (ex: all the DTOs representing the same *versioned* endpoint) shares a common interface.
Interfaces are used to recall a specific (or the nearest) version of the DTO.

!> NOTE: [dto management component](./dto-management.md) is needed for this kind of versioning.  
This needs a particular namespace structure to be created under an application sub-namespace (ex: `App\DTO`).<br><br>
Interfaces are often grouped under a generic namespace (ex: `App\DTO\Interfaces` or `App\DTO\Contracts`) while
versioned DTOs NEEDS to be placed under a namespace named as `{PREFIX}\v{MAJOR}\v{MAJOR}_{MINOR}` (ex: `App\DTO\v1\v1_0`).

An example:

In the version 1.0, my `User` is represented by fields `name` and `email`.  
In version 1.1, I want to add `phone` field to my `User` object (non-nullable field).

First of all I need to declare the common `UserInterface`:

```php
namespace App\DTO\Interfaces;
use App\Entity;

interface UserInterface {
    public function get(Entity\User $user): self;
}
```

Then I can create the two versions of the DTO.

```php
namespace App\DTO\v1\v1_0;
use App\DTO\Interfaces\UserInterface;
use App\Entity;

class User implements UserInterface
{
    public $name;
    public $email;

    public function get(Entity\User $user): self
    {
        $this->name = $user->getName();
        $this->email = $user->getEmail();

        return $this;
    }
}
```

```php
namespace App\DTO\v1\v1_1;
use App\DTO\Interfaces\UserInterface;
use App\Entity;

class User implements UserInterface
{
    public $name;
    public $email;
    public $phone;      // New field

    public function get(Entity\User $user): self
    {
        $this->name = $user->getName();
        $this->email = $user->getEmail();
        $this->phone = $user->getPhone();

        return $this;
    }
}
```

Now, with the `ServiceLocatorRegistry` and `Resolver` utilities from dto-management component, you can retrieve/create
a new instance of the `UserInterface` DTO just passing the version you want to retrieve to the resolver

```php
$serviceLocatorRegistry = \Solido\DtoManagement\Finder\ServiceLocatorRegistry::createFromNamespace('App\DTO');
$resolver = new \Solido\DtoManagement\InterfaceResolver\Resolver($serviceLocatorRegistry);

$resolver->resolve(UserInterface::class, '1.0');
```

The resolver will look into all the DTOs implementing `UserInterface`, calculates the version reading the namespace name
and creates a new instance of DTO with the nearest available version.

?> "Nearest" here means the exact version or the first available version reading backwards.<br><br>
Ex: if you requested version `3.0`, but the versions available are `1.0`, `1.2` and `3.1`, the `1.2` version will be selected.

?> This behavior could be discussed further to allow greater minor version (but same major) to be returned if
the exact version is not found in the tree. [Discuss the idea opening an issue on dto-management repository](https://github.com/solid-o/dto-management/issues)

## DTO extensions

Another interesting feature of DTOs is to be extensible via proxy and code generation.  
Via [dto-management component](./dto-management.md) you can create or register extensions to be applied to DTOs.

A DTO extension could add methods and properties or generate interceptors for existing
(and non-private) methods and properties.

[Data transformer component](./data-transformers.md) has one of these extensions which analyzes the DTO classes
via reflection and generates interceptors for transforming input data on property set or method call.

!> Generated code is only available while retrieving DTOs from a resolver. If you create a DTO instance with the `new`
keyword, the generated code will not be available as it is only present in the generated proxies.

### Writing a custom extension

DTO extensions must implement the `Solido\DtoManagement\Proxy\Extension\ExtensionInterface` interface and be registered
on `AccessInterceptorFactory` configuration ([check the documentation on how to register an extension](./dto-management.md?id=how-to-use-dto-extensions)).

The `ExtensionInterface` exposes only one method (`extend`) which accepts a `ProxyBuilder` object.  
The `ProxyBuilder` object is all what you need to add interfaces, traits, methods and properties to the generated proxy,
as well as add interceptors for methods and properties. Its `class` property gives you access to the `ReflectionClass`
object of the versioned DTO being extended and the `properties` attribute should help you to filter and iterate through
the DTO public and protected properties.

!> NOTE: Interceptors for private properties and methods cannot be created.

`ProxyBuilder` methods should be self-explanatory, but feel free to ask questions opening an issue on
dto-management repository. The responses will be included in this documentation to be helpful to other developers.

Through `ProxyBuilder` you can statically analyze the DTO class, search for attributes/annotations, load metadata
and generate public or protected methods, properties or interceptors for public and protected properties or methods.

?> Generated code will be checked for validity through `nikic/php-parser` library.
