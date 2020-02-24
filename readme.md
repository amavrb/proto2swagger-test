This produces a swagger file when run from the projects root folder.

`protoc --swagger_out=logtostderr=true,use_go_templates=true:. -I/Users/amavrb/git/googleapis -I. library/*/*.proto`

Where we can set flags by:
`--<tool_suffix>_out=<flags>:<path>`

Read more about the flags that can be set with the following commands
`protoc-gen-swagger --help`
or
`protoc-gen-grpc-gateway --help`

If you haven't recently updated `protoc_gen_swagger` run:

```bash
go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
```

Otherwise you could get a happy error message:
`Error while parsing option value for "openapiv2_operation": Message type "_swagger.options.Response" has no field named "examples".`

Examples are based on the `library.proto` that can be found [here](https://github.com/googleapis/googleapis/blob/master/google/example/library/v1/library.proto).

## Usage

The generation tool has options that are built-in in `protoc-gen-swagger`. Usage are as follows:

```java
option (grpc.gateway.protoc_gen_swagger.options.[kind of option]) = {
  ...
}
```

Availiable options are:
- `openapiv2_swagger`
  - File options
- `openapiv2_operation`
  - Method options
- `openapiv2_schema`
  - Message options
- `openapiv2_tag`
  - Service option
- `openapiv2_field`
  - Field options

## Support testing and results

### File Options

Should go directly beneath the import and option statements at the top level.
These options set properties of the swagger file.

```java
option (grpc.gateway.protoc_gen_swagger.options.openapiv2_swagger) = {
  info: {
    title: "Library Example API";
    version: "1.0";
    description: "Awesome API description right here. Could be longer, could be shorter."
  };
  security_definitions: {
    security: {
      key: "Bearer";
      value: {
        type: TYPE_API_KEY;
        name: "Authorization";
        in: IN_HEADER
      }
    }
  }
  security: { //Global security
    security_requirement: {
      key: "Bearer";
      value: {};
    }
  }
  host: "us.book.api.sinch.com"
  base_path: "/library";
  extensions: {
    key: "x-explorer-enabled"
    value {
      bool_value: true;
    }
  }
  extensions: {
    key: "x-samples-enabled"
    value {
      bool_value: true;
    }
  }
};
```

| Key                  | Proto | Result OAS2                        | Result readme                           | Ok  | Notes                                                                         |
| :--:                 | :---: | :---------:                        | :-----------:                           | :-: | --                                                                            |
| info                 | -     | ![key info](./images/info_key.png) | ![key result](./images/info_readme.png) |     | If used, description needs to be set. Otherwise shows as undefined in readme. |
| security_definitions | -     | ![](./images/sec-def-key.png)      | See next                             |     |                                                                               |
| security             | -     | ![](./images/sec-key.png)          | ![](./images/sec-result.png)            |     | Does not work well - looking at the example code shows `--header 'authorization: 3543645645'`, should be `--header 'authorization: Bearer 3543645645'`. |
| host and basepath    | -     | ![](./images/host-bp-key.png)      | ![](./images/host-bp-result.png)        |     |                                                                               |

Any important ones missing?

### Method Options

#### Tags

```java
rpc GetShelf(GetShelfRequest) returns (Shelf) {
  option (google.api.http) = {
    get: "/v1/{name}"
  };
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {
    tags: "Shelves";
  };
}
```

| Key | Proto | Result OAS2 | Result readme |  Ok | Notes |
| :--: | :---: | :---------: | :-----------: | :-: | -- |
| tags   | - | ![](./images/tags-key.png)  | ![](./images/tags-res.png) | Y | |

#### Responses

```java
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  option (google.api.http) = {
    get: "/v1/shelves/{name}/books"
  };
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {
    tags: "Books";
    responses: {
      key: "200";
      value: {
        examples: {
          key: "application/json"
          value: '{"books": [{"name": "books/34324","author": "Unknows artist","title": "The dancing flower","read": false},{"name": "books/34334354","author": "Artist","title": "The dancing","read": false}],"next_page_token":"3fsafe3af"}'
        }
      }
    }
    responses: {
      key: "503";
      value: {
        description: "Returned when the resource is temp. unavailable.";
      }
    }
  };
}
```

| Key | Code | Result OAS2 | Result readme |  Ok | Notes |
| :--: | :---: | :---------: | :-----------: | :-: | -- |
| responses | 200 | ![](./images/response-200.png) | ![](./images/response-200-res1.png) |   |   |
| responses   | 200 | See above  |  ![](./images/response-201-res2.png) |   |   |
| responses   | 503 |  ![](./images/response-503.png) | ![](./images/response-503-res.png)  |   |   |

#### Others

```java
rpc MoveBook(MoveBookRequest) returns (Book) {
  option (google.api.http) = {
    post: "/v1/shelves/books/{name}:move"
    body: "*"
  };
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {
    tags: "Books";
    deprecated: true;
    summary: "Move"
    description: "Description MoveBook"; //Overwrites description in proto
    external_docs: {
      url: "https://developers.sinch.com/docs";
      description: "Find out more MoveBook";
    }
  };
}
```

| Key           | Proto | Result OAS2                                     | Result readme                        | Ok  | Notes                                    |
| :--:          | :---: | :---------:                                     | :-----------:                        | :-: | --                                       |
| deprecated    | -     | As expected                                     | No indication of it being deprecated | N   |                                          |
| description   | -     | As expected but overwrites description in proto |                                      |     | Should probably use description in proto |
| external_docs | -     | Shows in OAS2                                   | Is not shown in readme               |     |                                          |
| summary       | -     | Overwrites summary set in proto |                                      |     |                                          |

Probably better to use the proto to set summary and description.

```java
// Move book <-- set summary in OAS2
//
// Moves a book to another shelf, and returns the new book. The book
// id of the new book may not be the same as the original book. <-- description in OAS 2
```

### Message Options

#### No options

```java
// A book
//
// A single book in the library.
message Book {
  option (google.api.resource) = {
    type: "library-example.googleapis.com/Book",
    pattern: "shelves/{shelf_id}/books/{book_id}"
  };
  // The resource name of the book.
  // Book names have the form `shelves/{shelf_id}/books/{book_id}`.
  // The name is ignored when creating a book.
  string book_id = 1;

  // The name of the book author.
  string author = 2;

  // The title of the book.
  string title = 3;

  // Value indicating whether the book has been read.
  bool read = 4;
}
```

| Key | Proto | Result OAS2 | Result readme |  Ok | Notes |
| :--: | :---: | :---------: | :-----------: | :-: | -- |
|  title | A book |   | ![](./images/book-s-v1-res.png)  |   |   |
| description   | A single book in the library  |   | See above  |   |   |

#### Set title, description and required fields

```java
// A book
//
// A single book in the library.
message Book {
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_schema) = {
    json_schema: {
        title: "The book"
        description: "Sets a new description in the swagger. Overwrites the previous one"
        required: ["book_id", "title"]
    }
};
  option (google.api.resource) = {
    type: "library-example.googleapis.com/Book",
    pattern: "shelves/{shelf_id}/books/{book_id}"
  };
  // The resource name of the book.
  // Book names have the form `shelves/{shelf_id}/books/{book_id}`.
  // The name is ignored when creating a book.
  string book_id = 1;

  // The name of the book author.
  string author = 2;

  // The title of the book.
  string title = 3;

  // Value indicating whether the book has been read.
  bool read = 4;
}
```

| Key         | Proto                     | Result OAS2 | Result readme                   | Ok  | Notes                                      |
| :--:        | :---:                     | :---------: | :-----------:                   | :-: | --                                         |
| title       | The book                  | -           | ![](./images/book-s-v2-res.png) |     | Should use proto to set title, not options |
| description | Sets a new description... | -           | See above                       |     | Should use proto to set title, not options |
| required    | -                         | -           | See above                       |     | Currently seems to be the best way to set required fields. |

#### external docs

```java
// A book
//
// A single book in the library.
message Book {
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_schema) = {
    external_docs: {
			url: "https://developers.sinch.com/docs";
			description: "Find out more about Book";
		}
};
  option (google.api.resource) = {
    type: "library-example.googleapis.com/Book",
    pattern: "shelves/{shelf_id}/books/{book_id}"
  };
  // The resource name of the book.
  // Book names have the form `shelves/{shelf_id}/books/{book_id}`.
  // The name is ignored when creating a book.
  string book_id = 1;

  // The name of the book author.
  string author = 2;

  // The title of the book.
  string title = 3;

  // Value indicating whether the book has been read.
  bool read = 4;
}
```
| Key          | Proto | Result OAS2 | Result readme | Ok  | Notes |
| :--:         | :---: | :---------: | :-----------: | :-: | --    |
| externalDocs | -     | As expected | Does not show | N   |       |

#### example

```java
// A book
//
// A single book in the library.
message Book {
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_schema) = {
    example: {
      value: '{"book_id": "3894jksla8HSa", "author": "Martin L. King", "title": "A few words about...", "read": "false"}'
    }
};
  option (google.api.resource) = {
    type: "library-example.googleapis.com/Book",
    pattern: "shelves/{shelf_id}/books/{book_id}"
  };
  // The resource name of the book.
  // Book names have the form `shelves/{shelf_id}/books/{book_id}`.
  // The name is ignored when creating a book.
  string book_id = 1;

  // The name of the book author.
  string author = 2;

  // The title of the book.
  string title = 3;

  // Value indicating whether the book has been read.
  bool read = 4;
}
```

| Key     | Proto | Result OAS2 | Result readme                  | Ok  | Notes          |
| :--:    | :---: | :---------: | :-----------:                  | :-: | --             |
| example |       | As expected | Not shown, shows in Swagger UI |     | ![](./images/book-ex-swui.png) |

### Service Options

These should work, but they are not in the generated swagger file.

```java
service LibraryService {
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_tag) = {
    description: "This will overwrite the description of the service in OAS2"
    external_docs: {
      url: "developers.sinch.com"
      description: "Find out more about Sinch find a number in a book! It's awsome!!"
    }
  };
  ...
  ...
}
```

| Key        | Proto | Result OAS2 | Result readme | Ok  | Notes |
| :--:        | :---: | :---------: | :-----------: | :-: | --    |
| **missing** | -     | -           | -             | -   |- |

### Field Options

#### required and description

```java
// A book
//
// A single book in the library.
message Book {
  ...
  // The resource name of the book.
  // Book names have the form `shelves/{shelf_id}/books/{book_id}`.
  // The name is ignored when creating a book.
  string book_id = 1 [
    (grpc.gateway.protoc_gen_swagger.options.openapiv2_field) = {
      description: "Resource name of the book. Overwrites description from proto"
      required: ['book_id']
    }
  ];

  // The name of the book author.
  string author = 2;

  // The title of the book.
  string title = 3;

  // Value indicating whether the book has been read.
  bool read = 4;
}
```

| Key | Proto | Result OAS2 | Result readme |  Ok | Notes |
| :--: | :---: | :---------: | :-----------: | :-: | -- |
| required   |   |   |   |  N  | Is not supported for properties in OAS2/OAS3 |
| description   | |   |   | Y  |   |

#### array minItems, maxItems

```java
// Request message for LibraryService.CreateShelf.
message CreateShelfRequest {
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_schema) = {
    json_schema: {
        required: ["shelf"]
    }
};
  // The shelf to create.
  repeated Shelf shelf = 1[(grpc.gateway.protoc_gen_swagger.options.openapiv2_field) = {
      min_items: 1,
      max_items: 400
    }];

  // First category for the shelf.
  Category firstCategory = 2;
}
```

| Key | Proto | Result OAS2 | Result readme |  Ok | Notes |
| :--: | :---: | :---------: | :-----------: | :-: | -- |
|  minItems/maxItems  |   | ![](./images/array-min-max-res.png)  |   |  N  | min and max items are not shown at all in readme. |
|    | |   |   |   |   |

### Bonus

#### Auto generated table
