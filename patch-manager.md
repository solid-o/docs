# Patch manager

Handles PATCH request. Supports JSON patch and merge-patch.

## Installation

```shell
$ composer require solido/patch-manager
```

## How it works

Handling a PATCH request can be cumbersome; the patch manager component makes this process simple.

### One method, two ways of patching

There are two different standards about patching resources:

- JSON patch: [RFC 6902](https://tools.ietf.org/html/rfc6902)
- Merge patch: [RFC 7386](https://tools.ietf.org/html/rfc7386)

The first one is the default patch method, identified by Content-Type "application/json-patch+json"
(but "application/json" will work too), and requires a document to describe how to modify the resource
through a set of operations to apply.

The second one is a more common operation and must be identified by Content-Type "application/merge-patch+json" header.  
It allows values setting just passing the modified fields (and values) in the request body.

### To be or not to be merge-patchable?

To be handled by `PatchManager` a resource must implement the `Solido\PatchManager\PatchableInterface`.  
This one enables the JSON patch ONLY. This allows you to disable direct value overwriting, avoiding missing update
situations allowing only add and remove operations on a specific field.

To be merge-patchable, a resource must implement `Solido\PatchManager\MergeablePatchableInterface` which is an extension
of `PatchableInterface`.  
This method leverages Symfony form component to bind (and validate) request data to the resource which will call
setter methods or will write to the property directly (if public).

?> In most cases `MergeablePatchableInterface` is what you need for your resources. `PatchableInterface` should be
used carefully and is only useful on some edge cases.

### Example

```php
use Solido\PatchManager\MergeablePatchableInterface;

class MyDTO implements MergeablePatchableInterface {
    public $name;
    public $email;
    public $phone;

    public function get(MyEntity $entity): self {
        ... // Populate this DTO
    }

    public function getTypeClass(): string {
        return MyDTOType::class;    // This should be configured to contain patchable fields
    }

    public function commit(): void {
        ... // Set modified values into entity and flush modifications
    }
}
```

You need to create the associated form type:

```php
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class MyDTOType extends AbstractType {
    public function buildForm(FormBuilderInterface $builder, array $options): void {
        $builder->add('name')
                ->add('email')
                ->add('phone');
    }

    public function configureOptions(OptionsResolver $resolver): void {
        $resolver->setDefaults([
            'data_class' => MyDTO::class,
        ]);
    }
}
```

?> Tip: you can use the same form type for resource creation, handling a POST request.

Then call the patch manager passing in the PATCH request.

```php
$request = \Symfony\Component\HttpFoundation\Request::createFromGlobals();
$patchManager = new \Solido\PatchManager\PatchManager();

$dto = ... // Retrieve data from db and populate the dto
$patchManager->patch($dto, $request);       // This analyzes request content and headers and applies the correct patch method.
```

If a validation error occurs while applying a patch document, an instance of 
`Solido\PatchManager\Exception\BadRequestException` will be thrown.
