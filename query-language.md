# Query language

Powerful query language for REST API lists.

## Installation

```shell
$ composer require solido/query-language
```

## How it works

Filtering list in a clear and flexible way upon a REST API is a difficult task which caused more than one headache
to any developer who addressed the problem.

Solido provides its own solution exposing a powerful yet flexible query language which is designed to be 
used through URL query parameters: lists define fields by which they can be filtered. Each field can be passed
as query parameter where the value can be a simple value or an operator.

Let's see an example:

Users list defined `name`, `email` and `birth_year` as filterable fields.

```
GET /users                      # Will return the complete list, unfiltered
GET /users?name=alessandro      # Will return the list of users whose name is exactly equal to "alessandro"
GET /users?email=$like(gmail)   # Will return the list of users containing "gmail" (case-insensitive) in the "email" field
GET /users?birth_year=$or(1990, $gte(2000))    # Birth year can be exacly 1990 or >= 2000
```

All the operators starts with a dollar sign ($) and the argument(s) are enclosed in parentheses.

!> To escape the dollar sign or a parenthesis, the symbol must be preceded by a backslash (`\`).

### List of operators

- `$all()`: no-op, does not apply any filter actually
- `$exists()`: field must be not-null
- `$not($expression)`: negates the argument expression
- `$eq(value)`: equals, field must be exactly equal to `value`. This operator is implicitly applied when passing a string to a filter
- `$neq(value)`: not equals, shortcut for `$not($eq(value))`
- `$like(value)`: matches when `value` is contained case-insensitively into the specified field
- `$gt(value)` and `$gte(value)`: _greater than_ and _greater than or equal_, used to compare numeric or date values
- `$lt(value)` and `$lte(value)`: _less than_ and _less than or equal_, same as above
- `$range(lower, upper)`: matches when the value is contained in the *inclusive* range `[lower, upper]`
- `$and(...)`: matches when all the expressions contained match
- `$or(...)`: matches when one of the expressions contained match
- `$in(value1, value2, ...)`: matches when value is exactly equals to one of the values passed
- `$entry(subfield, expression)`: used when navigating relations. Matches when the subfield of the field matches the expression

There is another special operator which can be used only on a special field:  
`$order(field, direction)`: can be used only on order filter.

Its use, however, is discouraged: an `X-Order` header should be passed to the request instead.

## Query language processor

Filters are declared and processed by processor objects.  
You can use one of provided doctrine processors or implement a new instance of `Solido\QueryLanguage\Processor\AbstractProcessor`

```php
$formFactory = \Symfony\Component\Form\Forms::createFormFactory();

// Processor here is a subclass of \Solido\QueryLanguage\Processor\AbstractProcessor
$processor = new Processor($formFactory, [
    'default_page_size' => 10,
    'max_page_size' => 30,
]);

$processor
    ->addField('name')
    ->addField('email')
    ->addField('birth_year');

$result = $processor->processRequest($request);
if ($result instanceof \Symfony\Component\Form\FormInterface) {
    // Filters contain invalid/unparsable data
    ...
} else {
    assert($result instanceof \Iterator);
    ...
}
```

## Query language walker

Expressions are processed and validated by tree walkers (instance of `\Solido\QueryLanguage\Walker\TreeWalkerInterface`).  
If custom query rules should be applied for a filter, a new tree walker should be created and used in the filter.

Tree walkers exposes a set of methods to iteratively walk into a filter expression building query expressions or
validating passed values in the process.

An example of filter walker (based on doctrine orm walker):

```php
class UserWalker extends \Solido\QueryLanguage\Walker\Doctrine\DqlWalker {
    public function walkComparison(string $operator, ValueExpression $expression) {
        $value = $expression->getValue();
        if ($value === 'me') {
            $expression = ValueExpression::create((string) $this->getCurrentUserId());
        }

        return parent::walkComparison($operator, $expression);
    }

    public function getCurrentUserId() {
        ...
    }
}
```

Validation walkers should extend `\Solido\QueryLanguage\Walker\Validation\ValidationWalker` calling `addViolation`
method when an invalid value is encountered.

An example of validation walker:

```php
class UuidValidationWalker extends \Solido\QueryLanguage\Walker\Validation\ValidationWalker {
    public function walkLiteral(LiteralExpression $expression) {
        $expressionValue = (string) $expression->getValue();
        if (! preg_match('/^[0-9A-F]{8}-[0-9A-F]{4}-4[0-9A-F]{3}-[89AB][0-9A-F]{3}-[0-9A-F]{12}$/i', $expressionValue)) {
            return;
        }

        $this->addViolation('{{ expression_value }} is not a valid uuid.', [
            '{{ expression_value }}' => (string) $expressionValue,
        ]);
    }
}
```

## Doctrine processors

Instead of implementing a new subclass of abstract processor, a provided processor for one of doctrine object manager
(or DBAL) could be used.  
A query builder must be created, initialized and passed as first argument of processor constructor.

Available processors:

| Processor Class                                           | Doctrine Project |
| --------------------------------------------------------- | ---------------- |
| `Solido\QueryLanguage\Processor\Doctrine\ORM\Processor`   | ORM              |
| `Solido\QueryLanguage\Processor\Doctrine\PhpCr\Processor` | PhpCr ODM        |
| `Solido\QueryLanguage\Processor\Doctrine\DBAL\Processor`  | DBAL             |

?> Processors for MongoDB ODM and Elastica ODM will be implemented in a future release

```php
$queryBuilder = $entityManager->getRepository(User::class)->createQueryBuilder('u');
$formFactory = \Symfony\Component\Form\Forms::createFormFactory();

// Processor here is a subclass of \Solido\QueryLanguage\Processor\AbstractProcessor
$processor = new Processor($queryBuilder, $formFactory, [
    'default_page_size' => 10,
    'max_page_size' => 30,
]);
```

### Field options

Doctrine processor fields accepts an array of options as second parameter of `addField`:

- `field_name` (defaults to filter name) allow overwriting the field name for entity or document
- `walker`: override filter walker
- `validation_walker`: override validation walker

`walker` and `validation_walker` accept a FQCN of the walker or a callable accepting three parameters:
the query builder to add the built query expression, the database field name and the field type

```php
$processor->addField('birth_year', [
    'walker' => static fn (QueryBuilder $queryBuilder, string $fieldName, string $fieldType) =>
        new BirthYearWalker($queryBuilder, $fieldName, $fieldType)
    'validation_walker' => BirthYearValidationWalker::class,
]);
```
