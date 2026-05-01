---
name: graphql
description: Design and implement GraphQL APIs — schema design, resolvers, Apollo Client, subscriptions, N+1 prevention, and federation
---

## What I do

I design and implement GraphQL APIs:

- **Schema design** — Types, interfaces, unions, enums, input types
- **Resolvers** — Queries, mutations, subscriptions, field resolvers
- **Client integration** — Apollo Client, Relay, cache management
- **Performance** — DataLoader for N+1, query complexity analysis
- **Real-time** — Subscriptions with WebSockets
- **Security** — Authentication, authorization, depth limiting, cost analysis

## When to use me

Use this skill when:
- Designing a new GraphQL API from scratch
- Implementing resolvers for a schema
- Setting up Apollo Client or Relay in a frontend
- Optimizing GraphQL query performance (N+1 problem)
- Adding real-time subscriptions
- Implementing authentication/authorization in GraphQL

## Schema design

### Type definitions

```graphql
type User {
  id: ID!
  email: String!
  name: String!
  avatar: String
  role: UserRole!
  posts: [Post!]! @auth(requires: OWNER)
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
  tags: [String!]!
  createdAt: DateTime!
}

enum UserRole {
  USER
  ADMIN
  MODERATOR
}

union SearchResult = User | Post

type Query {
  me: User!
  user(id: ID!): User
  users(limit: Int = 20, offset: Int = 0): [User!]!
  post(id: ID!): Post
  posts(filter: PostFilter, pagination: PaginationInput): PostConnection!
  search(query: String!): [SearchResult!]!
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post!
  deletePost(id: ID!): Boolean!
}

type Subscription {
  postCreated: Post!
  postUpdated(id: ID!): Post!
}

input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]!
}

input PostFilter {
  published: Boolean
  authorId: ID
  tags: [String!]
}

input PaginationInput {
  first: Int = 20
  after: String
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}

scalar DateTime
```

## Resolvers

### Apollo Server implementation

```ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const resolvers = {
  Query: {
    me: (_parent, _args, { user }) => {
      if (!user) throw new Error('Not authenticated');
      return db.user.findUnique({ where: { id: user.id } });
    },

    user: async (_parent, { id }: { id: string }) => {
      return db.user.findUnique({ where: { id } });
    },

    posts: async (_parent, { filter, pagination }: { filter?: PostFilter; pagination?: PaginationInput }) => {
      const { first = 20, after } = pagination ?? {};
      const cursor = after ? { id: after } : undefined;

      const posts = await db.post.findMany({
        where: {
          published: filter?.published,
          authorId: filter?.authorId,
          tags: filter?.tags ? { hasEvery: filter.tags } : undefined,
        },
        take: first + 1,
        cursor,
        skip: after ? 1 : 0,
        orderBy: { createdAt: 'desc' },
      });

      const hasNextPage = posts.length > first;
      const edges = hasNextPage ? posts.slice(0, -1) : posts;

      return {
        edges: edges.map(post => ({
          node: post,
          cursor: post.id,
        })),
        pageInfo: {
          hasNextPage,
          endCursor: edges[edges.length - 1]?.id ?? null,
        },
      };
    },
  },

  Mutation: {
    createPost: async (_parent, { input }: { input: CreatePostInput }, { user }) => {
      if (!user) throw new Error('Not authenticated');

      return db.post.create({
        data: {
          ...input,
          authorId: user.id,
        },
      });
    },
  },

  User: {
    posts: async (parent: User) => {
      return db.post.findMany({ where: { authorId: parent.id } });
    },
  },

  SearchResult: {
    __resolveType(obj: any) {
      if (obj.email) return 'User';
      if (obj.title) return 'Post';
      return null;
    },
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    const user = token ? await verifyToken(token) : null;
    return { user };
  },
});
```

## N+1 problem and DataLoader

```ts
import DataLoader from 'dataloader';

// Create loaders per request
function createLoaders() {
  return {
    userLoader: new DataLoader<string, User | null>(async (ids) => {
      const users = await db.user.findMany({
        where: { id: { in: [...ids] } },
      });

      const userMap = new Map(users.map(u => [u.id, u]));
      return ids.map(id => userMap.get(id) ?? null);
    }),

    postsByAuthorLoader: new DataLoader<string, Post[]>(async (authorIds) => {
      const posts = await db.post.findMany({
        where: { authorId: { in: [...authorIds] } },
      });

      const postsMap = new Map<string, Post[]>();
      for (const post of posts) {
        const existing = postsMap.get(post.authorId) ?? [];
        existing.push(post);
        postsMap.set(post.authorId, existing);
      }

      return authorIds.map(id => postsMap.get(id) ?? []);
    }),
  };
}

// Usage in resolvers
const resolvers = {
  Post: {
    author: (post: Post, _args: any, { loaders }: Context) => {
      return loaders.userLoader.load(post.authorId);
    },
  },

  User: {
    posts: (user: User, _args: any, { loaders }: Context) => {
      return loaders.postsByAuthorLoader.load(user.id);
    },
  },
};
```

## Apollo Client

### Client setup

```ts
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
});

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

export const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          posts: {
            keyArgs: ['filter'],
            merge(existing, incoming) {
              if (!existing) return incoming;
              return {
                ...incoming,
                edges: [...existing.edges, ...incoming.edges],
              };
            },
          },
        },
      },
    },
  }),
});
```

### React hooks

```tsx
import { useQuery, useMutation, gql } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts($first: Int!, $after: String) {
    posts(pagination: { first: $first, after: $after }) {
      edges {
        node {
          id
          title
          content
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
    }
  }
`;

function PostsList() {
  const { data, loading, error, fetchMore } = useQuery(GET_POSTS, {
    variables: { first: 20 },
  });

  const [createPost] = useMutation(CREATE_POST, {
    update(cache, { data }) {
      cache.modify({
        fields: {
          posts(existingPosts = []) {
            const newPostRef = cache.writeFragment({
              data: data.createPost,
              fragment: gql`
                fragment NewPost on Post {
                  id
                  title
                  content
                }
              `,
            });
            return [newPostRef, ...existingPosts];
          },
        },
      });
    },
  });

  if (loading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      {data.posts.edges.map(({ node }) => (
        <PostCard key={node.id} post={node} />
      ))}
      {data.posts.pageInfo.hasNextPage && (
        <button onClick={() => fetchMore({ variables: { after: data.posts.pageInfo.endCursor } })}>
          Load More
        </button>
      )}
    </div>
  );
}
```

## Quality checklist

- [ ] Schema designed with pagination (cursor-based) from the start
- [ ] DataLoader used to prevent N+1 queries
- [ ] Input validation on all mutations
- [ ] Authentication context passed to resolvers
- [ ] Field-level authorization with directives or resolver checks
- [ ] Query depth limiting to prevent DoS
- [ ] Error handling doesn't leak internal details
- [ ] Subscriptions properly authenticated
- [ ] Cache policies configured for Apollo Client
- [ ] Type generation from schema (GraphQL Code Generator)

## Anti-patterns I avoid

- Exposing database models directly as GraphQL types
- Not using DataLoader — N+1 queries kill performance
- Returning unbounded lists without pagination
- Missing input validation on mutations
- Leaking internal error details to clients
- Not handling authentication in the context
- Deeply nested queries without depth limiting
- Over-fetching fields the client doesn't need
- Not versioning schema changes carefully
- Using GraphQL for file uploads without multipart spec