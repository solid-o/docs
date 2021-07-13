# Changelog

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
