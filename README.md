# API Response Guideline

Adapting https://github.com/omniti-labs/jsend (with slight adjustments).

### Success Response

| Key | Presence | Type | Value |
| - | - | - | - |
| status | required | string | `"success"` |
| data | required | object | A free-form object `{}` |

Examples:

- Single resource

    ```json
    {
      "status": "success",
      "data": {
        "id": 1,
        "title": "A blog post"
      }
    }
    ```

- Different resources

    ```json
    {
      "status": "success",
      "data": {
        "post": {
          "id": 1,
          "title": "A blog post"
        },
        "comments_count": 1
      }
    }
    ```

    ```json
    {
      "status": "success",
      "data": {
        "post": {
          "id": 1,
          "title": "A blog post"
        },
        "comments": [
          {
            "id": 1,
            "body": "Good article!"
          }
        ]
      }
    }
    ```

- Collection

    Nests the collection under a key of its resource plural name (as opposed to
    `data` being an array). Array of post objects is set on `"posts"` key in
    this case. This way we can avoid breaking changes if, for example,
    pagination added later in the future.

    ```json
    {
      "status": "success",
      "data": {
        "posts": [
          {
            "id": 1,
            "title": "A blog post"
          },
          {
            "id": 2,
            "title": "Another blog post"
          }
        ]
      }
    }
    ```

- Collection and metadata

    ```json
    {
      "status": "success",
      "data": {
        "posts": [
          {
            "id": 1,
            "title": "A blog post"
          },
          {
            "id": 2,
            "title": "Another blog post"
          }
        ],
        "pagination": {
          "current_page": 1,
          "total_pages": 3,
          "count": 2,
          "total_count": 5
        }
      }
    }
    ```

- No data

    ```json
    {
      "status": "success",
      "data": {}
    }
    ```

### Failure Response

Treats client errors as failures (i.e. parameter missing, invalid auth token,
resource not found, validation errors, etc.). Ideally returned with 4xx HTTP
status code.

| Key | Presence | Type | Value |
| - | - | - | - |
| status | required | string | `"failure"` |
| data | required | object | A free-form object `{}` |

Examples:

```json
{
  "status": "failure",
  "data": {
    "message": "Authentication failed"
  }
}
```

```json
{
  "status": "failure",
  "data": {
    "first_name": "name can't be blank",
    "last_name": "is too long"
  }
}
```

```json
{
  "status": "failure",
  "data": {
    "code": 1003,
    "messages": [
      "First name can't be blank",
      "Last name is too long"
    ]
  }
}
```

### Error Response

Treats server errors/malfunctions as errors (i.e. redis connection issue, syntax
error, etc.). Ideally returned with 5xx HTTP status code.

| Key | Presence | Type | Value |
| - | - | - | - |
| status | required | string | `"error"` |
| message | required | string | Reason (i.e. `"Internal server error"`) |
| code | optional | integer | App error code (i.e. `1001`) |
| data | optional | object | A free-form object `{}` |

Examples:

- Simple message

    ```json
    {
      "status": "error",
      "message": "Internal server error"
    }
    ```

- With optional `code`

    ```json
    {
      "status": "error",
      "message": "Internal server error",
      "code": 5004
    }
    ```

- With optional `data`

    ```json
    {
      "status": "error",
      "message": "Internal server error",
      "data": {
        "backtrace": [
          "/path/to/script.rb:51:in `foo'",
          "/path/to/script.rb:47:in `foo_bar'"
        ]
      }
    }
    ```

- With optional `code` and `data`

    ```json
    {
      "status": "error",
      "message": "Internal server error",
      "code": 5004,
      "data": {
        "backtrace": [
          "/path/to/script.rb:51:in `foo'",
          "/path/to/script.rb:47:in `foo_bar'"
        ]
      }
    }
    ```
