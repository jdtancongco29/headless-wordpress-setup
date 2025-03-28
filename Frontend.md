# Front-End Setup
---

## Step 2: Set Up Next.js Frontend
### 2.1 Create Next.js App
```sh
npx create-next-app headless-training
cd headless-training
```

### 2.2 Install Dependencies
```sh
npm install graphql-request jwt-decode next-auth
```

### 2.3 Fetch Data from WPGraphQL
Create a utility file `src/lib/graphql.js`:
```js
import { request, gql } from 'graphql-request';

const WP_GRAPHQL_URL = 'https://yourwordpress.com/graphql';

export const getPosts = async () => {
  const query = gql`
    query {
      posts {
        nodes {
          id
          title
          content
        }
      }
    }
  `;
  return request(WP_GRAPHQL_URL, query);
};
```
### 2.4 Implement SSR in Next.js
In `pages/index.js`:
```js
import { getPosts } from '../lib/graphql';

export async function getServerSideProps() {
  const data = await getPosts();
  return { props: { posts: data.posts.nodes } };
}

export default function Home({ posts }) {
  return (
    <div>
      {posts.map(post => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: post.content }} />
        </div>
      ))}
    </div>
  );
}
```

### 2.5 Implement JWT Authentication
In `lib/auth.js`:
```js
export const login = async (username, password) => {
  const res = await fetch('https://yourwordpress.com/wp-json/jwt-auth/v1/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password }),
  });

  const data = await res.json();
  return data.token;
};
```

### 2.6 Secure API Routes in Next.js
In `pages/api/protected.js`:
```js
import jwt from 'jsonwebtoken';

export default function handler(req, res) {
  const { authorization } = req.headers;
  
  if (!authorization) {
    return res.status(401).json({ message: 'Unauthorized' });
  }

  try {
    const decoded = jwt.verify(authorization, 'your-secret-key');
    res.json({ message: 'Protected data', user: decoded });
  } catch (error) {
    res.status(401).json({ message: 'Invalid token' });
  }
}
```

---

## Step 3: Deploy
### 3.1 Deploy WordPress Backend
- Use **cPanel**, **DigitalOcean**, or **Kinsta**.
- Enable **HTTPS** for security.

### 3.2 Deploy Next.js
- Use **Vercel**:
  ```sh
  vercel
  ```
- Set environment variables in `.env.local`:
  ```
  WP_GRAPHQL_URL=https://yourwordpress.com/graphql
  ```

---
