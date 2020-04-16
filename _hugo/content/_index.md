+++
title = "My presentation"
outputs = ["Reveal"]
+++

# Sysl
## System Modeling Language Toolkit

---

## Swagger vs proto vs Sysl
- All specify APIs
- Sysl specifies how those APIs interact with other systems

---
## Swagger example
```yaml
definitions:
  todosResponse:
    format: tuple
    properties:
      completed:
        type: boolean
      id:
        format: integer
        type: number
      title:
        format: string
        type: string
      userId:
        format: integer
        type: number
    type: object
info:
  version: 0.0.0
paths:
  /todos/{id}:
    get:
      consumes:
      - application/json
      parameters:
      - format: integer
        in: path
        name: id
        type: number
      produces:
      - application/json
      responses: {}
swagger: "2.0"
```
--- 
# Proto example

```proto
message Request {
    string query = 1;
}

message Response {
    string query = 1;
}

service Foo{
    rpc thisEndpoint(Request) returns(Response);
}

service Bar{
    rpc AnotherEndpoint(Request) returns(Response);
}
```

---

## Sysl example
```go
Bar:
    AnotherEndpoint(input <: Foo.Request):
        return Response
Foo:
    thisEndpoint(input <: Request):
        Bar <- AnotherEndpoint
        return ok <: Response
    !type Request:
        query <: string
    !type Response:
        query <: string

```

---
## Modelling interactions

![Sequence Diagram Generation](images/sequenceDiagram.png?raw=true)


---
## Code Generation
- github.com/anz-bank/sysltemplate

---

## Specification
```go
simple:
    !type Stuff:
        Content <: string:
            @json_tag = "Content"  

    /foobar:
        GET:
            return ok <: Bar
    !type Bar:
        string
```

---
## Import a dependency from swagger

```yaml
definitions:
  todosResponse:
    format: tuple
    properties:
      completed:
        type: boolean
      id:
        format: integer
        type: number
      title:
        format: string
        type: string
      userId:
        format: integer
        type: number
    type: object
info:
  version: 0.0.0
paths:
  /todos/{id}:
    get:
      consumes:
      - application/json
      parameters:
      - format: integer
        in: path
        name: id
        type: number
      produces:
      - application/json
      responses: {}
swagger: "2.0"


```
---

## Calling a dependency

```go
simple:
    !type Stuff:
        Content <: string:
            @json_tag = "Content"  

    /foobar:
        GET:
            jsonplaceholder <- GET /todos/{id} // Call dep
            return ok <: Stuff
```


---

## Code Generation

- [Makefile](https://github.com/anz-bank/sysltemplate/blob/master/Makefile)
- Implement server
- Write function for `GET /foobar` endpoint
---

## Makefile
```Makefile
all: sysl

input = simple.sysl
app = simple
down = jsonplaceholder # this can be a list separated by a space or left empty
out = gen
# Current go import path This needs to change if you're not in the current go.mod directory

basepath = $(shell cat $(goModLocation) | grep module | sed 's/module//' | sed -e 's/^[[:space:]]*//')
```
---

## Writing our endpoint function

```go
// GetFoobarList refers to the endpoint in our sysl file
func GetFoobarList(ctx context.Context, req *simple.GetFoobarListRequest, client simple.GetFoobarListClient) (*simple.Response, error){
	return &simple.Response{Content:"Hello World"}, nil
}
```
---

## Implementing Server
```go
// implementation/server.go

// server.go contains all the manual config code that is used to implement the generated sysl
package server

var serverAddress = ":8080"

func LoadServices(ctx context.Context) error {
	router := chi.NewRouter()

	// simpleServiceInterface is the struct which is composed of our functions we wrote in `methods.go`
	// Struct embedding is used for the Service interface (yes, not interfaces)
	simpleServiceInterface := simple.ServiceInterface{
		GetFoobarList: GetFoobarList,
	}

	// Default callback behaviour
	genCallbacks := defaultcallback.DefaultCallback()

	serviceHandler := simple.NewServiceHandler(genCallbacks,
		&simpleServiceInterface,
		jsonplaceholder.NewClient(http.DefaultClient, "http://jsonplaceholder.typicode.com"))

	// Service Router
	serviceRouter := simple.NewServiceRouter(genCallbacks, serviceHandler)
	serviceRouter.WireRoutes(ctx, router)

	log.Println("Starting Server on " + serverAddress)
	log.Fatal(http.ListenAndServe(serverAddress, router))
	return nil
}
```
---


```curl localhost:8080/foobar```

```{"content":"Hello World"}```

---
## Calling a dependency
```go
func GetFoobarList(ctx context.Context, req *simple.GetFoobarListRequest, client simple.GetFoobarListClient) (*simple.Response, error) {
	response, err := client.GetTodos(ctx, &jsonplaceholder.GetTodosRequest{ID: 1})

	if err != nil {
		return nil, err
	}
	fmt.Println(response)
	return &simple.Response{Content: response.Title}, nil
}
```
---

```go
{"content":"delectus aut autem"}
```

- [Actual API called](http://jsonplaceholder.typicode.com/todos/1)

---

## What else do I get?

- Swaggerui/Redoc attached to handler
- Sequence diagrams
- Easily change API specification with minimal changes to code

---