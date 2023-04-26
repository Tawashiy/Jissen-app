---
theme: "white"
---

<style type="text/css"> p,h1,li { text-align: left; }
</style>

## GraphQL とコード自動生成機能

---

## Applo client とは？

- GraphQL API をシンプルにクライアント側で操作するためのライブラリ
- データの取得や操作のために便利な hooks を持つ
- キャッシュ機能や状態管理機能をもつ
- React、Vue だけではなく Svelte などの他のフレームワークで利用可能

---

### どのように使用するのか？

例）React で Applo Client の **useQuery** hooks を利用してデータを取得する

・スキーマを定義

```
#schema.graphql
type Post {
  id: String!
  title: String!
  email: String!
}

type Query{
  posts: Post[]
}
```

・実装

```
import { gql, useQuery } from '@apollo/client';

// レスポンスの型定義
interface PostQuery {
  posts: {
    id: string
    title: string
    email: string
  }[]
}

// 実行するクエリを記載
const postsQueryDocument = gql`
  query posts {
    posts {
      id
      title
      email
    }
  }
`

const Posts = () => {
  const { data } = useQuery<PostQuery>(postsQueryDocument);
   data.posts.map((post) => (
  　  //・・・
   ))}
}
```

---

### 型定義はめんどくさい

---

### GraphQL Code Generator

- スキーマとクエリ用のドキュメントを入力として与えることで、さまざまな言語とライブラリに対応した型情報を生成するツール（スキーマファーストの場合）
- スキーマファーストまたはコードファーストのアプローチを選択できる
- GraphQL スキーマの変更に伴うクライアントの再生成、GraphQL スキーマとクライアントコードの自動更新などの機能をもつ

---

### どのように使用するのか？

・スキーマを定義

```
#schema.graphql
type Post {
  id: String!
  title: String!
  email: String!
}

type Query{
  posts: Post[]
}
```

・クエリを定義

```
#posts.graphql
query Posts {
    posts {
      id
      title
      email
    }
  }
```

---

・設定ファイルを作成

```
#codegen.yml
schema: schema.graphql #スキーマファイルのパスを指定
documents: graphql/**/*.graphql #クエリの定義が含まれるドキュメントファイルのパターンを指定
generates:
  graphql/generated/graphql.tsx: # 生成されるコードのファイルパス
    plugins:  #使用されるプラグイン
      - typescript
      - typescript-operations
      - typescript-react-apollo
    config:
      withHooks: true
```

codegen を実行すると...
↓

---

```
// ./graphql/generated
export type PostsQueryVariables = Exact<{ [key: string]: never; }>;

// 型定義
export type PostsQuery = { __typename?: 'Query', posts?: Array<{ __typename?: 'Post', id: string, title: string, email: string } | null> | null };


// GraphQLクエリ
export const PostsDocument = gql`
  query Posts {
    posts {
      id
      title
      email
    }
  }
`;

/**
 * __usePostsQuery__
 *
 * To run a query within a React component, call `usePostsQuery` and pass it any options that fit your needs.
 * When your component renders, `usePostsQuery` returns an object from Apollo Client that contains loading, error, and data properties
 * you can use to render your UI.
 *
 * @param baseOptions options that will be passed into the query, supported options are listed on: https://www.apollographql.com/docs/react/api/react-hooks/#options;
 *
 * @example
 * const { data, loading, error } = usePostsQuery({
 *   variables: {
 *   },
 * });
 */

// hooks
export function usePostsQuery(baseOptions?: Apollo.QueryHookOptions<PostsQuery, PostsQueryVariables>) {
        const options = {...defaultOptions, ...baseOptions}
        return Apollo.useQuery<PostsQuery, PostsQueryVariables>(PostsDocument, options);
      }
export function usePostsLazyQuery(baseOptions?: Apollo.LazyQueryHookOptions<PostsQuery, PostsQueryVariables>) {
          const options = {...defaultOptions, ...baseOptions}
          return Apollo.useLazyQuery<PostsQuery, PostsQueryVariables>(PostsDocument, options);
        }
export type PostsQueryHookResult = ReturnType<typeof usePostsQuery>;
export type PostsLazyQueryHookResult = ReturnType<typeof usePostsLazyQuery>;
export type PostsQueryResult = Apollo.QueryResult<PostsQuery, PostsQueryVariables>;
```

### 使用例

```
import { usePostsQuery } from './graphql/generated';

const Posts = () => {
  const { data } = usePostsQuery();

  // ...
}
```

---

### 比較

```
# Applo clientのhooksを使用
const { data } = useQuery<PostQuery>(postsQueryDocument);

# GraphQL Code Generatorにより生成されたhooksを使用
const { data } = usePostsQuery();

```

---

## まとめ

- 複数の API を実行して、必要な情報を取得するために、フロントで成形する必要がないので、フロントのコードが綺麗になる
- 型定義や hooks の自動生成も可能なので、変更による手間やタイプミスの心配がない
