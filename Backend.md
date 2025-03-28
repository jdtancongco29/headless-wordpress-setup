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

