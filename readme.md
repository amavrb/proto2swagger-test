This produces a swagger file when run from the projects root folder.

`protoc --swagger_out=logtostderr=true,json_names_for_fields=true:. -I/Users/path/to/googleapis -I. library/*/*.proto`

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
