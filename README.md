# Thinkmill GraphQL Style Guide

This style guide documents the standards we have developed for designing GraphQL Schemas at Thinkmill.

- Terms/Concepts/Dictionary
- Workflow and Process (GraphQL as Design System equivalent)
  - https://twitter.com/JedWatson/status/1170867029659791366
  - GraphQL as ideal abstraction layer for the business schema
- Types
  - Schema Overview
  - Field type conventions
    - ID
    - String
    - Number
    - Boolean
    - Enum
    - Date
  - Relationships
- Queries
  - Standard Arguments
  - Filtering
  - Pagination
  - Sorting
  - Recursion of patterns/Nest Queries(?)
- Mutations
- Custom Types/Queries/Mutations (non-CRUD)
- Authentication
- Validation and Error Handling
- Caching
- Permissions

## Terms

In order to keep the style guide implementation-agnostic, we refer to _entities_, _entity types_ and _entity collections_.

In an SQL-backed implementation, the _entity type_ for a `User` would be the schema; the _entity collection_ would be the `users` table and an `entity` would be a record in the table.

Each _entity type_ has a singular and plural label, which we refer to as `{Entity}` and `{Entities}` in this style guide. Each entity also contains _fields_, which map to properties of the entity.

Throughout the guide, we will use the following example schema:

```
type User {
  id: ID!
  name: String
  company: Company
  dob: String
  status: UserStatusEnum
  emails: [Email]
}

enum UserStatusEnum {
  ACTIVE
  INACTIVE
}

type Company {
  id: ID!
  name: String
  employeeCount: Number
  users: [User]
}

type Email {
  id: ID!
  email: String
  isVerified: Boolean
  user: User
}
```

## Workflow and Process

// TODO

- How do you come up with a schema?
- System requirements -> schema
- Identify the queries/mutations you need and use this to inform the data schema
- How do we get UX designers to think about the queries they need?
- A graphQL schema can be easily related across the development spectrum
- Given a UX design, how do you identify the schema requirements (in the same way you'd identify the design system requirements)

## Queries

For each entity type we generate the four top level queries:

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

JED WHAT DO WE HAVE TO SAY ABOUT \_EntityMeta? it returns a count that is always 1?

## Field types

JED: Better name than field types?

## Relationship

### `where` filters

- `{relatedEntity}_every`: whereInput
- `{relatedEntity}_some`: whereInput
- `{relatedEntity}_none`: whereInput
- `{relatedEntity}_is_null`: Boolean

## String

### `where` filters

- `{Field}:` String
- `{Field}_not`: String
- `{Field}_contains`: String
- `{Field}_not_contains`: String
- `{Field}_starts_with`: String
- `{Field}_not_starts_with`: String
- `{Field}_ends_with`: String
- `{Field}_not_ends_with`: String
- `{Field}_i`: String
- `{Field}_not_i`: String
- `{Field}_contains_i`: String
- `{Field}_not_contains_i`: String
- `{Field}_starts_with_i`: String
- `{Field}_not_starts_with_i`: String
- `{Field}_ends_with_i`: String
- `{Field}_not_ends_with_i`: String
- `{Field}_in`: [String]
- `{Field}_not_in`: [String]

## Operators

ToDo: Where does this section belong?

- `AND`: [TaskWhereInput]
- `OR`: [TaskWhereInput]

## ID

- `{Field}`: ID
- `{Field}_not`: ID
- `{Field}_in`: [ID!]
- `{Field}_not_in`: [ID!]

## Integer

- `{Field}: Int`
- `{Field}_not`: Int
- `{Field}_lt`: Int
- `{Field}_lte`: Int
- `{Field}_gt`: Int
- `{Field}_gte`: Int
- `{Field}_in`: [Int]
- `{Field}_not_in`: [Int]

## Mutations
