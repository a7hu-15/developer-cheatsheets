# 📐 GraphQL & Schema Design Reference Cheatsheet

A developer reference guide for GraphQL Schema Definition Language (SDL), query/mutation syntax, fragments, directives, resolver execution, N+1 query optimization with DataLoader, and API security.

---

## 🏗️ 1. Schema Definition Language (SDL)

### Basic Types & Scalars
```graphql
scalar DateTime

enum Role {
  ADMIN
  MEMBER
  GUEST
}

type User {
  id: ID!
  username: String!
  email: String!
  role: Role!
  createdAt: DateTime!
  posts(limit: Int = 10): [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}
```

### Interfaces & Unions
```graphql
# Interface: Abstract type enforcing shared fields
interface Node {
  id: ID!
  createdAt: DateTime!
}

type Article implements Node {
  id: ID!
  createdAt: DateTime!
  headline: String!
}

type Video implements Node {
  id: ID!
  createdAt: DateTime!
  durationSeconds: Int!
}

# Union: Choice between unrelated types
union SearchResult = Article | Video | User
```

### Inputs & Mutations
```graphql
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  deletePost(id: ID!): Boolean!
}
```

---

## 🔍 2. Querying & Operations

### Queries with Aliases & Variables
```graphql
query GetUserProfiles($userId: ID!, $includePosts: Boolean!) {
  primaryUser: user(id: $userId) {
    username
    email
    posts(limit: 5) @include(if: $includePosts) {
      id
      title
    }
  }
}
```

### Inline Fragments & Unions
```graphql
query SearchContent($query: String!) {
  search(query: $query) {
    __typename
    ... on Article {
      headline
    }
    ... on Video {
      durationSeconds
    }
    ... on User {
      username
    }
  }
}
```

---

## ⚡ 3. The N+1 Problem & DataLoader Solution

### The N+1 Query Anti-Pattern
Without batching, fetching $N$ posts with their author executes $1$ query for posts + $N$ queries for authors:
```
Query 1: SELECT * FROM posts LIMIT 10;
Query 2..11: SELECT * FROM users WHERE id = author_id;  (Executed N times!)
```

### DataLoader Batching Pattern (Node.js / Python)
DataLoader collapses individual database lookups into a single batched `IN` query:

```javascript
// DataLoader batches IDs within a single event-loop tick
const userLoader = new DataLoader(async (userIds) => {
  const users = await db.users.findMany({ where: { id: { in: userIds } } });
  const userMap = new Map(users.map(u => [u.id, u]));
  return userIds.map(id => userMap.get(id));
});

// Resolver delegates to DataLoader
const resolvers = {
  Post: {
    author: (post, args, context) => context.userLoader.load(post.authorId),
  },
};
```
```
Batched Query: SELECT * FROM users WHERE id IN (1, 2, 3, 4, 5, ...);
```

---

## 🌐 4. Relay Cursor Connection Specification (Pagination)

Standard GraphQL pagination pattern for infinite scroll:

```graphql
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type PostEdge {
  cursor: String!
  node: Post!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type Query {
  posts(first: Int, after: String): PostConnection!
}
```

---

## 🛡️ 5. GraphQL Security & Best Practices

| Concern | Vulnerability | Mitigation Strategy |
|---------|---------------|--------------------|
| **Nested Depth Attacks** | `user { friends { friends { friends ... } } }` | Enforce maximum query depth limit (e.g. max depth = 6). |
| **Query Complexity** | Heavy nested fields consuming server CPU/memory | Calculate field complexity score and cap per request. |
| **Introspection Disclosure** | Exposing API schema to external attackers | Disable `__schema` introspection in Production environments. |
| **Rate Limiting** | Automated token exhaustion | Enforce query-cost bucket rate limiting instead of IP request counts. |

---
