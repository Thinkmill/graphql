# Thinkmill graphQL styleguide

ToDo: Define up-front:

- What is an "Entity"

## Queries

For each entity we generate the four top level queries:

- `all{Entities}`
- `_all{Entities}Meta`
- `{Entity}`
- `_{Entity}Meta`

### `all{Entities}`

```gql
query {
  allUsers {
    id
  }
}
```

Retrieves all entities from a entity collection. `all{Entities}` queries accept a standard list of query arguments. These are:

### `where`

```gql
query {
    allUsers (where: { id_starts_with_i: 'A'} ) {
        id
    }
}
```

Limit results to those matching the where clause. Where clauses generated will depend on the field types available.

### `search`

Will search the entity collection to limit results. Search logic is defined by the entity type.

```gql
query {
  allUsers(search: "Mike") {
    id
  }
}
```

### `orderBy`

```gql
query {
  allUsers(orderBy: "name") {
    id
  }
}
```

Will order the results. The orderBy string should match a field name in the entity collection.

### `first`

```gql
query {
  allUsers(first: 10) {
    id
  }
}
```

Select this many results from the entity collection, sorted by the `orderBy` argument and matching the `where` and `search` values. Works like specifying `TOP` in SQL.

If less results are available, the number of available results will be returned.

### `skip`

```gql
query {
  allUsers(skip: 10) {
    id
  }
}
```

Skip the number of results specified, before returning the number of results specified by the `first` argument, sorted by the `orderBy` argument and matching the `where` and `search` values.

If the value of `skip` is greater than the number of available results, zero results will be returned.

### Combining query arguments for pagination

When `first` and `skip` are used together with the `count` from `_all{Entities}Meta`, this is sufficient to implement pagination of an entity collection.

It is important to provide the same `where` and `search` arguments to both the `all{Entities}` and `_all{Entities}Meta` queries. For example:

```gql
query {
  allUsers (search:'a', skip: 10, first: 10) {
    id
  }
  _allUsersMeta(search: 'a') {
    count
  }
}

```

When `first` and `skip` are used together, skip works as an offset for the `first` argument. For example`(skip:10, first:10)` selects results 11 through 20.

Both `skip` and `first` respect the values of the `where`, `search` and `orderBy` arguments.

## `_all{Entities}Meta`

```gql
query {
  _allUsersMeta {
    count
  }
}
```

Get the total number of entities matching any `where` and `search` clauses provided.

The `where` and `search` clauses work the same as the `all{Entities}` query.

## `{Entity}`

```gql
query {
  User(where: { id: ID }) {
    name
  }
}
```

Retrieves an entity from a collection. The single entity query has one argument.

### `where`

```gql
query {
  User(where: { id: ID }) {
    name
  }
}
```

This where clause is different to the all entities where clause in that it only can only filter by a matching `id`.

## `_{Entity}Meta`

# Per Entity

### type {Entity} with `id` and {fields}

- ... scalar fields
- relationship fields
  - Add
    - {field}(query meta): Type or [Type] of related entity
    - \_{field}Meta(query meta): \_QueryMeta

### input {entity}(where)

all{Entity}()
