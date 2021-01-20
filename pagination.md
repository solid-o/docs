# Pagination

Endless pagination utilities.

## Installation

```shell
$ composer require solido/pagination
```

## How it works

`PagerIterator` is the core of the pagination component and provides the basic logic for an endless pagination
iterator implementation.

To provide an endless-pagination the collection must be ordered by two fields:

- the first one represents the main order (ex: creation timestamp, alphabetical name, etc)
- the second one is generally the object identifier and will be used to calculate the page checksum

This ordering is needed to build a coherent page token, which is the encoded equivalent 
of "where to start for the next page".

The token contains three information:

- The encoded representation of the first order field (ex: timestamp) of the last element
- The offset of the object relative to similar objects with the same timestamp
- The crc32 checksum of the second order field of all objects in the page

The page token is part of page slice calculation in pager iterator.  
The algorithm works this way:

- The collection is sorted by the main order and checksum field.
- If the collection is empty, an empty page is returned.
- If a token is not set, the first slice (0 to page size) is returned.
- In every other case, we must filter the objects array, which contains every item in the database.<br>
  We use the timestamp inside the token as a reference, compared with the first field of the orderBy array.
- If there's a difference in timestamps or checksums the fallback procedure is:
  - Return the first `pageSize` elements of the filtered array
  - If checksum and timestamp are ok the procedure, return the first `pageSize` elements of the filtered 
    array, taking the offset into account

```php
$objects = [ ... ];
$iterator = new \Solido\Pagination\PagerIterator($objects, [
    'createdAt' => 'DESC',
    'id' => 'ASC',
]);

$iterator->setPageSize(25);  // Default page size is 10
$page = \iterator_to_array($iterator);   // Returns the first slice of 25 elements

$token = $iterator->getNextPageToken();  // Gets the next page token
$iterator->setPageToken($token);

$secondPage = \iterator_to_array($iterator);  // Returns the second page of 25 elements
```

## Doctrine pagers

!> `refugis/doctrine-extra` library must be installed to work with doctrine pagers.

While the `PagerIterator` can page every iterable collection, it is convenient to fire limited
queries to the databases.

That's why the pagination component contains specific `PagerIterator`s for doctrine ORM, doctrine phpcr ODM,
doctrine DBAL and refugis elastica ODM.  
These pagers use the iterators provided by `refugis/doctrine-extra` library which handles the trickiest parts
of creating object iterators, enable caches and so on. Also, it provides a normalized version of the iterators
which can be used as base for the various `PagerIterator`s.

Whether you use doctrine ORM, PhpCr ODM or doctrine DBAL, you have to pass a query builder with filters
already set as the first argument of the correct `PagerIterator` class and the orderings as the second argument.

The iterator will then build the correct filter and limit clauses and do a filtered query based on
the passed page token.

!> You should ensure that the correct indexes are in place to allow the filter query to run at maximum speed.<br>
For example you probably want to add an index to the `created_at` column of your table if this is the main 
ordering field as the resulting query will contain a `WHERE created_at >= ?` and a `ORDER BY created_at` clauses.
