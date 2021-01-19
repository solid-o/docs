# Data transformers

Transforms request data to another representation

## Installation

```shell
$ composer require solido/data-transformers
```

## How it works

REST APIs communicates via HTTP protocol. Because of this all the data sent to the API are textual representation
of the real data, but most of the time it is simply uncomfortable to deal with textual data.  
Additionally, transformation from text to a PHP object is often a repetitive task.

All the transformers implements [`Solido\DataTransformers\TransformerInterface`](https://github.com/solid-o/data-transformers/blob/master/src/TransformerInterface.php)
which exposes only one method: `transform`.

The `transform` method simply transforms the data passed into another representation and throws 
`Solido\DataTransformers\Exception\TransformationFailedException` if transformation fails (ex: data are invalid).

## DTO integration

Data transformers express their full potential when paired with DTOs.  
As you can read in the [dto chapter](./dto.md) the DTOs are not simple containers of data, but can be
*enhanced* via extensions and proxy code generation.

Data transformers library provides one of this extension (in the `Solido\DataTransformers\TransformerExtension` class)
which enables the use of `#[Transform]` attribute (or annotation).

Using `#[Transform]` attribute on a public or protected property declared on a DTO, code to call the specified transformer
will be generated for every property write call.

For example, if I declare `MyDTO` as follows

```php
use Solido\DataTransformers\Transformer\DateTransformer;

class MyDTO implements MyDTOInterface
{
    /**
     * @var string
     */
    public $name;

    /**
     * @var DateTimeImmutable
     */
    #[Transform(DateTransformer::class)]
    public $birthDay;
}
```

And I recall the DTO from resolver

```php
$dto = $resolver->resolve(MyDTOInterface::class);
```

I can set a date on `birthDay` property simply setting a string into the property:

```php
$dto->birthDay = '1990-05-30';   // Magic happens...

var_dump($dto->birthDay);        // Dumps:
                                 // object(DateTimeImmutable) {
                                 //    ["date"] => "1990-05-30 00:00:00.000000"
                                 //    ...
                                 // }
```

#### Usage on methods

Additionally, the `#[Transform]` attribute works on single-argument methods, transforming the argument
before it reaches the body of the method.

?> In the future the single-argument limitation could be removed analyzing the parameters attributes,
but at the moment the compatibility with annotation (which can't be applied on parameters) has been retained.

## Builtin transformers

This library contains some common text-to-data transformers to be used when parsing request data.

#### Base64UriFileTransformer

Transforms a data-URI into an UploadedFile object.

Accepts: `string` (base64-encoded data file, ex: `data:image/gif;base64,R0lGODdhAQABAIAAAP///////ywAAAAAAQABAAACAkQBADs=`)<br>
Returns: instance of `Symfony\Component\HttpFoundation\File\UploadedFile`

#### BooleanTransformer

Transforms a boolean text representation into a bool value.

Accepts: `string` (one of `1`, `true`, `yes`, `on`, `y`, `t`, `0`, `false`, `no`, `off`, `n`, `f`)  
Returns: `bool`

#### DateTimeTransformer

Transforms an ISO8601 (or atom) string into an instance of DateTimeInterface.

Options (as constructor arguments):

`$outputTimezone` - Set the timezone to apply after the transformation has finished  
`$asImmutable` - Whether to return a DateTimeImmutable (default) or a mutable DateTime object.

Accepts: `string` (ISO8601 date time - YYYY-MM-DD'T'HH:mm:ssZ)  
Returns: `DateTimeInterface`

#### DateTransformer

Accepts: `string` (YYYY-MM-DD or DD/MM/YYYY)  
Returns: `DateTimeImmutable`

Time part is set to midnight.

#### IntegerTransformer

Transforms a numeric string into an integer value.

Accepts: `string`  
Returns: `int`

#### PhoneNumberTransformer

Transforms a phone number string into an instance of `libphonenumber\PhoneNumber`.

?> `giggsey/libphonenumber-for-php` library must be installed

Accepts: `string`  
Returns: `libphonenumber\PhoneNumber`

#### MoneyTransformer

Transforms a money representation into a Money object.

?> `moneyphp/money` library must be installed

Accepts: `string` (numeric) or `{ amount: string, currency: string }` array  
Returns: `Money\Money` object

!> If only a numeric value is passed, the currency is automatically set to `EUR`.

#### CurrencyTransformer

Transform a currency string into a Currency object.

?> `moneyphp/money` library must be installed

Accepts: `string`  
Returns: `Money\Currency` object

---

There are also some special transformers:

#### ChainTransformer

Applies multiple transformers one after another.

#### MappingTransformer and NullableMappingTransformer

Applies a transformer to all the elements of an array.  
The nullable variant skips null elements, while the other will try to convert also the null values.

#### PageTokenTransformer

Used in endless-pagination, converts a string into a PageToken representation. See 
[pagination component](./pagination.md) for further information.

#### UrnToItemTransformer

Use an urn-to-item converter to transform a textual urn into an entity/document. See
[urn section](./urn.md) for further information.

## Custom transformers

To create a custom transformer you only need to implement `Solido\DataTransformers\TransformerInterface` and expose
the `transform` method as per interface documentation

```php
interface TransformerInterface
{
    /**
     * Transforms a value to another representation.
     *
     * This method must be able to deal with empty values. Usually this will
     * be an empty string, but depending on your implementation other empty
     * values are possible as well (such as NULL). The reasoning behind
     * this is that value transformers must be chainable. If the
     * transform() method of the first value transformer outputs an
     * empty string, the second value transformer must be able to process that
     * value.
     *
     * By convention, transform() should return NULL if an empty string is passed.
     *
     * @param mixed $value The value in the transformed representation
     *
     * @return mixed The transformed value
     *
     * @throws TransformationFailedException when the transformation fails.
     */
    public function transform($value);
}
```
