# Symfony integration

Provides Solido components integration for the [Symfony framework](https://symfony.com/)

## Installation

```shell
$ composer require solido/symfony
```

bundles.php:
```php
return [
    ...
    Solido\Symfony\SolidoBundle::class => ['all' => true],
];
```

## Configuration

The symfony integration library exposes the configuration for all the installed components through the symfony
config component.

YAML Reference (and default values):

```yaml
solido:

  # Whether to enable test features
  test:                 false
  form:

    # Register form data mapper for all forms in the project
    register_data_mapper: false

    # Register the auto-submit extension for all forms
    auto_submit:          false

  # Enables the request processing features (Accept header parsing, versioning guessing)
  request:
    enabled:              true
    versioning:
      enabled:              true

      # Can be "accept", "custom_header" or a service id
      guesser:              accept

      # Required for custom header version guesser, specify the header to check
      custom_header_name:   X-Version

    # Set the default mime type if no Accept header is present on the request
    default_mime_type:    application/json

    # Sets the acceptable MIME types for Accept header
    priorities:

      # Defaults:
      - application/json
      - application/x-json
      - text/xml
      - application/xml
      - application/x-xml

  # Whether to enable or not the body converter component
  body_converter:
    enabled:              true

  # Enables the automatic serialization of views and data returned from controllers
  serializer:
    enabled:              true

    # The serializer id (must implement Solido\Serialization\SerializerInterface)
    id:                   ~
    charset:              UTF-8
    groups:

      # Default:
      - Default
    catch_exceptions:     true
  data_transformers:
    date_time:

      # Default output timezone for date time data transformer
      timezone:             null

  # Configure DTO management component and automatically register dto service locator registry
  dto:
    routing:

      # Whether to register the dto routing loader
      loader:               true

    # List of enhanced DTO namespaces
    namespaces:           [] # Required

    # List of interfaces excluded by dto resolvers
    exclude:              []
  security:
    action_listener:
      enabled:              false
      prefix:               ~
    policy_checker:
      enabled:              false
      service:              ~
  urn:
    enabled:              true

    # The URN default domain
    default_domain:       null
  cors:
    enabled:              true
    allow_credentials:    true
    allow_origin:

      # Default:
      - *
    allow_headers:        []
    expose_headers:       []
    max_age:              0
    paths:

      # Prototype
      -
        enabled:              true
        allow_credentials:    ~
        host:                 ~
        path:                 ~ # Required
        allow_origin:         []
        allow_headers:        []
        expose_headers:       []
        max_age:              ~
```

## Forms

In solido forms are used in place of deserialization.  
In a way, forms (as implemented in Symfony) are a powerful, event emitting, self-configurable and self-modifiable
deserializer which denormalizes request content into a given PHP object and collects eventual errors in the process.

In addition, if validation component is installed, it automatically calls validation and binds validation errors
to the correct sub-form.

### Data mapper

Unlike normal forms, forms in an API system do not need to bind data from object to the form for display.  
To avoid extra transformations and unneeded casts and conversions, a special data mapper should be registered to
the API only forms.

`Solido\Symfony\Form\OneWayDataMapper` has been provided and will be auto-registered if `solido.form.register_data_mapper`
configuration key is set to true.

### Auto submit

In symfony form is submitted only if at least one field is present in the request.
This obviously excludes validation of empty requests. The auto-submit extension will call `Form::submit` method
even if no field is present in the request, subsequently calling related data validation and error collection.

## Serialization

Serializer libraries will be automatically searched and the appropriate adapter will be registered
if one of the serializer supported by [serialization component](./serialization-component.md) is found.

The order in which the serializer libraries are searched is:

1. Kcs serializer (preferred)
2. JMS serializer
3. Symfony serializer

If one of these libraries is present and its services are registered in the service container, the
corresponding adapter will be set as default view serializer.

### View

To enable automatic serialization of value returned by the controller, the `#[Solido\Symfony\Annotation\View]`
attribute (or annotation) must be added to the controller method.

The view attribute allows to customize the HTTP status code, the serialization groups (even with a provider method)
and the serialization type.

## DTO

_Enhanced DTO_ interfaces are automatically registered as service and can be autowired in any service.

!> DTO services are registered as non-shared. Check the [Symfony Dependency Injection component
documentation](https://symfony.com/doc/current/service_container/shared.html) for more information.

### Routing loader

_Enhanced DTOs_ methods can be target of the framework routing component, avoiding the creation of a controller
which subsequently call a DTO method.

If `solido.dto.routing.loader` configuration key is set to `true` (default) you can add it to your `routing.yaml`
file, specifying which namespace should be checked for `#[Route]` attributes and annotations.

Example: (routing.yaml)

```yaml
app_dto:
    resource: App\DTO\
    type: dto_annotations
```

This will search for `#[Route]` attributes in all the DTOs `interfaces` found in `App\DTO` namespace.

Example:

```php
// src/DTO/Interfaces/UserInterface.php

use Symfony\Component\Routing\Annotation\Route;

interface UserInterface
{
    #[Route('/user/{id}', method: 'GET')]
    public function get(Entity\User $user): self;
}
```

This will register a route responding to `GET /user/{id}` requests.  
DTO routing loader implicitly sets `#[View]` attribute on controller.

## CORS

[CORS component configuration](./cors.md?id=configuration) is exposed under `solido.cors` key.

If enabled, request event listeners will be registered to automatically handle `OPTIONS` requests and to
add CORS headers to every response sent out by the framework.
