+++
title = "My presentation"
outputs = ["Reveal"]
+++

# Sysl
## System Modeling Language Toolkit


---
# Swagger vs proto vs Sysl
- All specify APIs
- Sysl specifies how those APIs interact with other systems

---
# Swagger example
```yaml
"Boiler plate server info here"
paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in CommonMark or HTML.
      responses:
        '200':    # status code
          description: A JSON array of user names
          content:
            application/json:
              schema: 
                type: array
                items: 
                  type: string
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

# Sysl example
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
# Modelling interactions

<!-- [Sequence Diagram Generation](http://anz-bank.github.io//sysl-playground/?input=QmFyOgogICAgQW5vdGhlckVuZHBvaW50KGlucHV0IDw6IEZvby5SZXF1ZXN0KToKICAgICAgICByZXR1cm4gUmVzcG9uc2UKRm9vOgogICAgdGhpc0VuZHBvaW50KGlucHV0IDw6IFJlcXVlc3QpOgogICAgICAgIEJhciA8LSBBbm90aGVyRW5kcG9pbnQKICAgICAgICByZXR1cm4gUmVzcG9uc2UKICAgICF0eXBlIFJlcXVlc3Q6CiAgICAgICAgcXVlcnkgPDogc3RyaW5nCiAgICAhdHlwZSBSZXNwb25zZToKICAgICAgICBxdWVyeSA8OiBzdHJpbmc=&cmd=c3lzbCBzZCAtbyAicHJvamVjdC5zdmciIC1zICJGb28gPC0gdGhpc0VuZHBvaW50IiB0bXAuc3lzbA==) -->

<!-- [image]
src = "/static/img/simple.svg" -->

![Sequence Diagram Generation](images/simple.svg?raw=true)


---
# Code Generation
- github.com/anz-bank/sysltemplate

---


