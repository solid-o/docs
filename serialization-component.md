# Serialization

Provides a shared interface and adapters to data serializers

## Installation

```shell
$ composer require solido/serialization
```

## How it works

This component provides adapters for serializers and exposes them under a shared interface.  
Additionally, provides two event listeners to call proxy initialization during serialization in JMS Serializer and Kcs Serializer.

It is mostly by framework integrations during response processing.

All the adapters implement `Solido\Serialization\SerializerInterface`, which exposes only the `serialize` method.  
The `serialize` method three parameters:

- `$data` the data to be serialized
- `$format` string representation of the target format (ex: `json`, `xml`, etc.)
- `$context` array of options
  - `groups`: string[] - serialization groups
  - `type`: string - override serialized type
  - `serialize_null`: bool - whether to serialize null values or skip them
  - `exable_max_depth`: bool - whether to enable max depth checks

The context options are respected as long as the underlying serializer supports them

Bundled adapters:

| Class                                                   | Serializer                                                                       |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `Solido\Serialization\Adapter\KcsSerializerAdapter`     | [Kcs Serializer](https://alekitto.github.io/serializer/) (recommended)           |
| `Solido\Serialization\Adapter\JmsSerializerAdapter`     | [JMS Serializer](https://jmsyst.com/libs/serializer)                             |
| `Solido\Serialization\Adapter\SymfonySerializerAdapter` | [Symfony Serializer](https://symfony.com/doc/current/components/serializer.html) |

```php
$serializer = Kcs\Serializer\SerializerBuilder::create()
                  ->configureListeners(function (Symfony\Component\EventDispatcher\EventDispatcherInterface $dispatcher) {
                      $dispatcher->addSubscriber(new Solido\Serialization\DTO\KcsSerializerProxySubscriber());
                  })
                  ->build();
$adapter = new Solido\Serialization\Adapter\KcsSerializerAdapter($serializer);

$dto = UserDTO::createFromData(...);
$adapter->serialize($dto, 'json', ['serialize_null' => true]);   // Returns a JSON string
```

A custom serializer adapter can be added implementing `Solido\Serialization\SerializerInterface`.

### Proxy pre-serialize listeners

If using _enhanced DTO_ like the ones described in [DTO section](./dto.md?id=dto-extensions), you must ensure that the
initialization methods has been called before the serialization starts.

That's why this component provides event subscribers that listen on pre-serialize event for JMS serializer
and Kcs serializer which call the initialization method if the data to be serialized is a DTO proxy.
