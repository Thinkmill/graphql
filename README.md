# Thinkmill GraphQL Style Guide

This style guide documents the standards we have developed for designing GraphQL Schemas at Thinkmill.

- Concepts
- Workflow and Process (GraphQL as Design System equivalent)
  - https://twitter.com/JedWatson/status/1170867029659791366
  - GraphQL as ideal abstraction layer for the business schema
- Types
  - Schema Overview
  - See [Schemas and Types](https://graphql.org/learn/schema/) in the GraphQL Spec
  - Field type conventions and use-cases
    - ID (Scalar)
    - String (Scalar)
    - Boolean (Scalar)
    - Number (Int or Float)
    - Enum (Not quite a scalar type)
    - Date (Not built in)
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

## Concepts

In order to keep the style guide implementation-agnostic, we refer to _entities_, _entity types_ and _entity collections_.

In an SQL-backed implementation, the _entity type_ for a `User` would be the schema; the _entity collection_ would be the `users` table and an `entity` would be a record in the table.

Each _entity type_ has a singular and plural label, which we refer to as `{Entity}` and `{Entities}` in this style guide. Each entity also contains _fields_, which map to properties of the entity.

We call entity properties _fields_, which have _field types_. Field types map to other entity types or scalar GraphQL Types.

Throughout the guide, we will use the following example schema:

```
scalar DateTime

enum PriorityEnum {
  LOW
  MEDIUM
  HIGH
}

type Todo {
  id: ID!
  name: String
  priority: PriorityEnum
  dueDate: DateTime
  user: User
}

type User {
  id: ID!
  name: String
  age: Int
  height: Float
  todos: [Todo]
}

Query {
  # Get one todo item
  Todo(id: ID!): Todo
  # Get all todo items
  allTodos: [Todo!]!
}

Mutation {
  addTodo(name: String!, priority: Priority = LOW): Todo!
  removeTodo(id: ID!): Todo!
}
```

```gql
query {
  allTodos {
    name
    users(first: 10) {
      name
    }
  }
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
- `{Entity}`
- `_all{Entities}Meta`
- `_{Entity}Meta`

#### `{Entity}`

```gql
query {
  User(where: { id: ID }) {
    name
  }
}
```

Retrieves an entity from a collection. The single entity query has one argument.

#### `all{Entities}`

```gql
query {
  allUsers {
    id
  }
}
```

Retrieves all entities from a collection.

#### `_all{Entities}Meta`

```gql
query {
  _allUsersMeta {
    count
  }
}
```

Get the total number of entities from a collection.

#### `_{Entity}Meta`

// TODO: This should probably be taken out of the spec

```gql
query {
  _UserMeta(where: { id: ID }) {
    count
  }
}
```

### Filtering

These apply to queries for the `all{Entities}` and `{all}EntitiesMeta` types

#### `where`

```gql
query {
    allUsers (where: { id_starts_with_i: 'A'} ) {
        id
    }
}
```

Limit results to those matching the where clause. Where clauses generated will depend on the field types available.

##### Operators

// TODO: Where does this section belong?

- `AND`: [UserWhereInput]
- `OR`: [UserWhereInput]

#### `search`

> Need to consider whether this should be included or whether it's just a KeystoneJS specific thing

Will search the entity collection to limit results. Search logic is defined by the entity type.

```gql
query {
  allUsers(search: "Mike") {
    id
  }
}
```

### Sorting

#### `orderBy`

```gql
query {
  allUsers(orderBy: "name_ASC") {
    id
  }
}
```

> Ascending vs Descending?

Will order the results. The orderBy string should match a field name in the entity collection.

### Pagination

#### `first`

```gql
query {
  allUsers(first: 10) {
    id
  }
}
```

Select this many results from the entity collection, sorted by the `orderBy` argument and matching the `where` and `search` values. Works like specifying `TOP` in SQL.

If less results are available, the number of available results will be returned.

#### `skip`

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

## Field types

### ID

- `{Field}`: ID
- `{Field}_not`: ID
- `{Field}_in`: [ID!]
- `{Field}_not_in`: [ID!]

### String

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

### Number

- `{Field}: Int`
- `{Field}_not`: Int
- `{Field}_lt`: Int
- `{Field}_lte`: Int
- `{Field}_gt`: Int
- `{Field}_gte`: Int
- `{Field}_in`: [Int]
- `{Field}_not_in`: [Int]

### Boolean

// TODO

### Enum

// TODO

### Date

// TODO

## Mutations

// TODO
