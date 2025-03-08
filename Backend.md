# Headless WordPress with JWT, GraphQL, ACF, and Custom APIs (Backend)

## Step 1: Set Up WordPress Backend
### 1.1 Install WordPress
- Install WordPress on a local or cloud server.
- Ensure you have a working database.

### 1.2 Install Required Plugins
- **WPGraphQL**: Provides GraphQL API.
- **WPGraphQL for ACF**: Enables ACF fields in GraphQL queries.
- **Advanced Custom Fields (ACF)**: For managing custom fields.
- **JWT Authentication for WP-API**: Enables JWT authentication.
- **WP REST API Custom Endpoints** (if needed for custom APIs).

### 1.3 Enable JWT Authentication
1. Add the following to `wp-config.php`:
   ```php
   define('JWT_AUTH_SECRET_KEY', 'your-secret-key');
   define('JWT_AUTH_CORS_ENABLE', true);
   ```
2. Test JWT by sending a request to:
   ```sh
   POST /wp-json/jwt-auth/v1/token
   ```
   with credentials (`username` and `password`).

### 1.4 Configure WPGraphQL
- Visit `GraphQL` settings in WP Admin.
- Enable access to custom post types.
- Test queries at `/graphql` endpoint.

### 1.5 Set Up ACF Fields
- Create a new field group in ACF.
- Enable GraphQL support in field settings.
- Query ACF fields in WPGraphQL.

### 1.6 Create Custom API Endpoints
- Use `register_rest_route` in `functions.php`:
  ```php
  add_action('rest_api_init', function () {
      register_rest_route('custom/v1', '/data/', array(
          'methods' => 'GET',
          'callback' => 'get_custom_data',
          'permission_callback' => '__return_true'
      ));
  });

  function get_custom_data() {
      return new WP_REST_Response(['message' => 'Hello from custom API'], 200);
  }
  ```
- Test via `/wp-json/custom/v1/data`.

---

## Step 2: Set Up Next.js Frontend
### 2.1 Create Next.js App
```sh
npx create-next-app@latest my-next-app
cd my-next-app
```

### 2.2 Install Dependencies
```sh
npm install graphql-request jwt-decode next-auth
```

### 2.3 Fetch Data from WPGraphQL
Create a utility file `lib/graphql.js`:
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

