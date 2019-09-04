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

Retrieves all entities from a entity collection. Each entity can accept a standard list of query arguments. These are:

### `where`

```gql
query {
    allUsers (where: { id_starts_with_i: 'A'} ) {
        id
    }
}
```

Limit results to those matching the where clause. Where clauses generated will depend on the field types available. See: [entityWhere]()

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
  allUsers(skip: 10, first: 10) {
    id
  }
  _allUsersMeta {
    count
  }
}
```

When `first` and `skip` are used together, the total `{first}` results will be selected _after_ skipping the first `{skip}` results, where both respect the values of the `where`, `search` and `orderBy` arguments.

# Per Entity

### type {Entity} with `id` and {fields}

- ... scalar fields
- relationship fields
  - Add
    - {field}(query meta): Type or [Type] of related entity
    - \_{field}Meta(query meta): \_QueryMeta

### input {entity}(where)

all{Entity}()
