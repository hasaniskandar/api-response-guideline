# API Response Guideline Proposal

Adapting https://github.com/omniti-labs/jsend.

### Success Response

| Key    | Required | Type   |                                        |
| ------ | -------- | ------ | -------------------------------------- |
| status | required | string | Always set to `"success"`              |
| data   | required | object | A free-form object `{ ... }` or `null` |

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
            "id": 456,
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

    **Please vote** :pray: :grin:

    Another ambiguity is the use of `200 OK` and `204 No Content` for `DELETE`
    request to the resource. Would be great if we can choose one and agree to
    stick with it. Choices I can think of are:

    - Use `200 OK` with `data` is set to `null`

        ```json
        {
          "status": "success",
          "data": null
        }
        ```

    - Use `200 OK` with `data` is set to empty object `{}`

        ```json
        {
          "status": "success",
          "data": {}
        }
        ```

    - Use `204 No Content`

### Failure Response

Treats client errors as failures (i.e. parameter missing, invalid auth token,
resource not found, validation errors, etc.). Ideally returned with 4xx HTTP
status code.

| Key    | Required | Type   |                              |
| ------ | -------- | ------ | ---------------------------- |
| status | required | string | Always set to `"failure"`    |
| data   | required | object | A free-form object `{ ... }` |

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

| Key     | Required | Type    |                                        |
| ------- | -------- | ------- | -------------------------------------- |
| status  | required | string  | Always set to `"error"`                |
| message | required | string  | Reason, e.g. `"Internal server error"` |
| code    | optional | integer | App error code, e.g. `1001`            |
| data    | optional | object  | A free-form object `{ ... }`           |

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
