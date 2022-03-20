# Changelog

### 0.3.0 (2022-02-28)

BC breaking changes:

- [Common] `AutoSubmitRequestHandler` has been moved to data mapper component
- [Query Language] Processors now use `DataMapperFactory` instead of symfony's `FormFactoryInterface`
- [Patch Manager] `MergeablePatchableInterface` has been renamed to `MergePatchableInterface`
- [Patch Manager] `MergePatchableInterface` does not expose `getTypeClass` anymore:
  has been replaced with `getDataMapper(): DataMapperInterface` which returns a data mapper object (from data mapper component)
- [Symfony] removed `form` configuration key. Not needed anymore
- [Symfony] `CollectionType` and `UnstructuredType` form types have been moved into data mapper component
- [Symfony] exceptions are now serialized as `application/problem+json`

Changes:

- [Api problem] Add Api problem component to serialize exceptions and errors in a standard way
- [Data mapper] Add Data Mapper component to abstract request data mapping to objects/DTOs
- [Patch Manager] rewritten property accessor internals
- [Symfony] Add data mapper integration
- [Symfony] Support symfony 6
- [Test utils] Functional requests testing helpers
- Multiple bugfixes

### 0.2.1 (2021-10-26)

Changes:

- [DTO Management] fix accessor scope binding: use custom public scope simulator code generator
- [DTO Management] removed dead code
- [Query language] abstract method `createField` removed from abstract processor
- [Symfony] change annotations alias names
- [Symfony] fix controller arguments resolution on DTO methods
- [Symfony] fix preload crash when security component is not installed
- [Symfony] fix error on container compilation when dto support is enabled but expression language component is missing
- [Symfony] add Lock DTO extension
- [Test utils] fix compatibility with `doctrine/cache` >= 2.0
- [Test utils] fix compatibility with `doctrine/dbal` >= 3.0

### 0.2.0 (2021-07-13)

BC breaking changes:

- [CORS] `solido/common` is a required dependency.
- [QueryLanguage] `AbstractProcessor::buildIterator` signature has been changed.
  Now it requires a `Query` object as second argument.
- [Test utils] `solido/common` is a required dependency for response constraints.

Changes:

- Allow usage of PSR-7 ServerRequest objects

Bugfix:

- [Data transformers] fix undefined variable error

### 0.1.2 (2021-07-04)

Changes:

- [DTO Management] add resolvers to autowire dtos when using the standalone component
- [Pagination] allow order by related object field
- [Symfony] support symfony 5.3
- [Symfony] allow custom expression language provider to be used in security configuration attribute

Bugfix:

- [Query language] fix name collision
- [Serialization] fix default serialization groups with symfony serializer
- [Symfony] fix cors on streamed response
