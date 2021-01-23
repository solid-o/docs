# DTO management

Manages, finds and enhance DTOs.

!> Please read the [DTO section](./dto.md) of this documentation to learn about the role
of DTOs in solido ecosystem.

## Installation

```shell
$ composer require solido/dto-management
```

## How it works

As described in the DTO section of this documentation, DTOs needs to be created and organized under the same
namespace, with a special `Interfaces` sub-namespace and versioned namespaces for the real data handling.

The central point of the DTO management is the `ServiceLocatorRegistry` which searches for the interfaces contained
into the `Interfaces` sub-namespace and calculates available versions for each model.

The `ServiceLocatorRegistry` also enables the proxy generation to allow DTO extensions to be built and used.

Once the `ServiceLocatorRegistry` has been created, you need a resolver to retrieve/create a new instance of the
DTO for a specific version. For this task you can create a new `Solido\DtoManagement\InterfaceResolver\Resolver`
passing the newly created `ServiceLocatorRegistry` as argument.

The resolver exposes two method: `has` and `resolve`. With the first one you can check if a given interface is present
on the DTO registry, the second one creates a new DTO instance for the interface and the nearest version you pass
as argument. By default, the version is set to 'latest'.

Let's see an example:

Folder structure (and namespaces):

```
src              (App\)
 ┗ DTO           (App\DTO\)
  ┗ Interfaces   (App\DTO\Interfaces\)
    ┗ ProductInterface.php
    ┗ UserInterface.php
  ┗ v1
    ┗ v1_0
      ┗ User.php
    ┗ v1_1
      ┗ User.php
      ┗ Product.php
  ┗ v2
    ┗ v2_0
      ┗ Product.php
```

You can then create a dto registry and an associated resolver with:

```php
$serviceLocatorRegistry = \Solido\DtoManagement\Finder\ServiceLocatorRegistry::createFromNamespace('App\DTO');
$resolver = new \Solido\DtoManagement\InterfaceResolver\Resolver($serviceLocatorRegistry);
```

You can now retrieve a specific version of a DTO with:

```php
$resolver->resolve(UserInterface::class, '1.0');  // Will return a new instance of App\DTO\v1\v1_0\User
$resolver->resolve(UserInterface::class);         // Will return a new instance of App\DTO\v1\v1_1\User (1.1 is the latest version of User)

$resolver->resolve(ProductInterface::class, '1.0');  // Will throw: Product does not exist in 1.0 nor in one of previous versions
$resolver->resolve(ProductInterface::class, '1.1');  // Will return a new instance of App\DTO\v1\v1_1\Product
$resolver->resolve(ProductInterface::class, '2.0');  // Will return a new instance of App\DTO\v2\v2_0\Product
$resolver->resolve(ProductInterface::class, 'latest');  // Will return a new instance of App\DTO\v2\v2_0\Product
```

If using the [versioning component](./versioning.md) the version will be set as `_version` attribute on the request
object. `Resolver` class accepts request objects also and extract the version to retrieve the correct DTO.

```php
assert($request instanceof Symfony\Component\HttpFoundation\Request);
$resolver->resolve(ProductInterface::class, $request);
```

## How to use DTO extensions

In order to use a DTO extension like the one contained in the [data transformers component](./data-transformers.md?id=dto-integration)
or a [custom one](./dto.md?id=writing-a-custom-extension) you need to pass a
correctly configured `AccessInterceptorFactory` as the `proxyFactory` parameter of `createFromNamespace`.

Example:

```php
$configuration = new \Solido\DtoManagement\Proxy\Factory\Configuration();
$configuration->addExtension(new \Solido\DataTransformers\TransformerExtension());

$proxyFactory = new \Solido\DtoManagement\Proxy\Factory\AccessInterceptorFactory($configuration);
$serviceLocatorRegistry = \Solido\DtoManagement\Finder\ServiceLocatorRegistry::createFromNamespace('App\DTO', proxyFactory: $proxyFactory);
```

## Proxy factory configuration

The `Solido\DtoManagement\Proxy\Factory\Configuration` is based on `ProxyManager\Configuration` of 
`ocramius/proxy-manager` package. Please refer to the package documentation for additional configuration.

It is however recommended writing the proxies to specific cache dir to avoid generation on every request.  
The DTO management component exposes a cache writer generation strategy for the proxy-manager which can be
used to write a proxy on the first generation and then load it from the cache.

```php
$configuration->setProxiesTargetDir(__DIR__.'/cache/dto_proxies')
              ->setGeneratorStrategy(new \Solido\DtoManagement\Proxy\GeneratorStrategy\CacheWriterGeneratorStrategy($configuration));
```
