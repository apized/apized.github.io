# Documentation (Work In Progress)

## Introduction

It is our belief that defining the [model](/documentation.md?id=model) is, in our not so humble opinion, both the
initial and one of the most important steps of designing a good solution. We also believe that if we're building pure
REST APIs, both the API and the DB model should be one and the same - with exceptions of course but the exceptions
confirm the rule. Any interactions should be derived from manipulating the state, i.e. the model, and we should have no
actions in our API since those are the HTTP actions.

In that spirit, apized takes exactly this approach. A project's initial and one of it's most important classes are the
models we define. By adding the `@Apized` annotation we are informing the framework that this is a central piece of our
model and should therefore be exposed in the form of a controller (and accompanying service and repository).

In the simplest case of a CRUD server this is all you would need. For more complex scenarios we add Behaviours, which
should encapsulate small self-contained pieces of business logic.

Sometimes a behaviour can be considered as a part of the service (logic shared between differnet behaviours) and for
these cases we allow extensions the service. Repositories can also be extended to allow for custom-made queries.

## Core concepts

### Model

All models must extend the `BaseModel` class which defines a set of default fields:

| Field         | Type          | Description                                                                          |
|---------------|---------------|--------------------------------------------------------------------------------------|
| id            | UUID          | This is defined as being a UUID and will be used as the primary key on the DB table. |
| version       | Long          | Used for optimistic locking.                                                         |
| createdBy     | UUID          | ID of the user that created the instance                                             |
| createdAt     | LocalDateTime | Date & time the instance was created at                                              |
| lastUpdatedBy | UUID          | ID of the user that last updated the instance                                        |
| lastUpdatedAt | LocalDateTime | Date & time of the last time this instance was updated                               |
| metadata      | UUID          | A JSON-like structure of Maps and Lists containing Primitive data types and Strings  |

The `createdBy`, `createdAt`, `lastUpdatedBy` and `lastUpdatedAt` fields provide a quick audit overview of the instance
while the `metadata` field stores data we might want to have associated with the model but is not part of our model.

```java

@Entity // (1)
@Serdeable // (2)
@Apized // (3)
public class Organization extends BaseModel {
  @NotBlank // (4)
  private String name;
}
```

> 1. `@Entity` marks the entity as being stored on the DB.

> 2. `@Serdeable` marks the entity as being serializable (API).

> 3. `@Apized` marks the entity as an apized entity.

> 4. `@NotBlank` javax.validation.constraints for your fields.

### Behaviour

### Extensions

### Context

## Enhanced REST Endpoints

With apized your endpoints offer quite a bit more that your standard REST endpoint. You can ask for what you need and
get exactly what you ask for, get multiple resources with a single request and make use of advanced filtering without
having
to write a single line of code beyond your model.

GET `/organizations/3fedcab7-97b7-4f81-b49f-2a70864f7cfa`
<!-- tabs:start -->

# **Response**

```json
{
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa",
  "name": "Org A",
  "employees": [
    "e9fa40a7-3044-4328-82e9-f710a0911452",
    "f0203a8f-cb3b-430b-a866-cdf7ea1ed730"
  ]
}
```

<!-- tabs:end -->

GET `/organizations/3fedcab7-97b7-4f81-b49f-2a70864f7cfa/employee/e9fa40a7-3044-4328-82e9-f710a0911452`
<!-- tabs:start -->

# **Response**

```json
{
  "id": "e9fa40a7-3044-4328-82e9-f710a0911452",
  "name": "Sen Eng",
  "age": 35
}
```

<!-- tabs:end -->

### Field filtering

No more over or under fetching of data, get exactly and only what you need.

GET `/organizations/3fedcab7-97b7-4f81-b49f-2a70864f7cfa?fields=name`
<!-- tabs:start -->

# **Response**

```json
{
  "name": "Org A",
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa"
}
```

<!-- tabs:end -->

### Model drilling

Get related data with one single request. We do model drilling for both queries and mutations.

GET `/organizations/3fedcab7-97b7-4f81-b49f-2a70864f7cfa?fields=name,employees.name`
<!-- tabs:start -->

# **Response**

```json
{
  "employees": [
    {
      "name": "Sen Eng",
      "id": "e9fa40a7-3044-4328-82e9-f710a0911452"
    },
    {
      "name": "Jun Eng",
      "id": "f0203a8f-cb3b-430b-a866-cdf7ea1ed730"
    }
  ],
  "name": "Org A",
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa"
}
```

<!-- tabs:end -->

### Smaller payloads

Send only what needs to change instead of having to send the whole object.

PUT `/organizations/3fedcab7-97b7-4f81-b49f-2a70864f7cfa?fields=name`
<!-- tabs:start -->

# **Request**

```json
{
  "name": "Organization A"
}
```

# **Response**

```json
{
  "name": "Organization A",
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa"
}
```

<!-- tabs:end -->

Which also works with model drilling.

PUT `/organizations/3fedcab7-97b7-4f81-b49f-2a70864f7cfa?fields=name,employees.name`
<!-- tabs:start -->

# **Request**

```json
{
  "name": "Organization A",
  "employees": [
    {
      "name": "John Smith",
      "id": "e9fa40a7-3044-4328-82e9-f710a0911452"
    },
    "f0203a8f-cb3b-430b-a866-cdf7ea1ed730"
  ]
}
```

# **Response**

```json
{
  "employees": [
    {
      "name": "John Smith",
      "id": "e9fa40a7-3044-4328-82e9-f710a0911452"
    },
    {
      "name": "Jun Eng",
      "id": "f0203a8f-cb3b-430b-a866-cdf7ea1ed730"
    }
  ],
  "name": "Organization A",
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa"
}
```

<!-- tabs:end -->

### Searching

Because apized understands your model you get querying out of the box for any field. This also extends to models you
might be fetching via model drilling.

GET `/organizations?search=name=Org%20A`
<!-- tabs:start -->

# **Response**

```json
{
  "employees": [
    "e9fa40a7-3044-4328-82e9-f710a0911452",
    "f0203a8f-cb3b-430b-a866-cdf7ea1ed730"
  ],
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa",
  "name": "Org A",
  "version": 0,
  "createdBy": "d568705a-a581-4923-828b-0f97c0891163",
  "createdAt": 1675363907394,
  "lastUpdatedBy": null,
  "lastUpdatedAt": 1675363907394,
  "metadata": {}
}
```

<!-- tabs:end -->

GET `/organizations?search=employee.name~=Sen`
<!-- tabs:start -->

# **Response**

```json
{
  "employees": [
    "e9fa40a7-3044-4328-82e9-f710a0911452",
    "f0203a8f-cb3b-430b-a866-cdf7ea1ed730"
  ],
  "id": "3fedcab7-97b7-4f81-b49f-2a70864f7cfa",
  "name": "Org A",
  "version": 0,
  "createdBy": "d568705a-a581-4923-828b-0f97c0891163",
  "createdAt": 1675363907394,
  "lastUpdatedBy": null,
  "lastUpdatedAt": 1675363907394,
  "metadata": {}
}
```

<!-- tabs:end -->

## Behaviours

Apized follows the Controller-Service-Repository pattern. These layers are auto-generated for you at
compile time. You can extend the generated service and repository layers by implementing extensions.
The behaviours form a request execution pipeline. Behaviours encapsulate a given business requirement in its own class
and leaves nice and isolated.

## Audit trail

By using apized, your model will automatically be put into an audit trail. This is achieved by storing a snapshot of the
object at the time of the interaction on the database. By default, the object shape is the exact same shape you would
get from calling the api itself. In order to provide flexibility you can use the `@AuditField` and `@AuditIgnore` to
manipulate what this snapshot will look like on your database.

## Event publishing

By using apized, your model will automatically be put into an ESB. This is achieved by generating a snapshot of the
object at the time of the interaction. By default, the object shape is the exact same shape you would
get from calling the api itself. In order to provide flexibility you can use the `@EventField` and `@EventIgnore` to
manipulate what this snapshot will look like on your ESB message.

## Testing

Testing is done with groovy & Spock for simplicity. Integratin testing also included with most of the building blocks in
place.

## Documentation

Our generated Controllers are annotated with OpenAPI annotations and therefore you get openapi documentation out of the
box.

## Security

Our security is based on permissions to allow our users to have control as coarse or fine-grained or coarse grained as
desired.

Permissions where designed to follow the following pattern [project].[model].[action].[id].[field].[value] where the
create action excludes the [id] section since there’s no id to target. These permissions support wildcards
`sample.organization.update.*.name` - this permission means the user can update the name of any organization.

Some examples:

| Permission                                                             | description                                                                                                                                                         |
|------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `*`                                                                    | God permission, can to anything on any service                                                                                                                      |
| `sample`                                                               | Access to do everything on anything in the sample service                                                                                                           |
| `sample.organization`                                                  | Can do all operations on all organizations                                                                                                                          |
| `sample.organization.create`                                           | Can create organizations                                                                                                                                            |
| `sample.organization.update.4b949872-f9ec-489f-8f4c-613b7a9c1ecf`      | Can update organization with id `4b949872-f9ec-489f-8f4c-613b7a9c1ecf`                                                                                              |
| `sample.organization.update.4b949872-f9ec-489f-8f4c-613b7a9c1ecf.name` | Can update only the name of organization `4b949872-f9ec-489f-8f4c-613b7a9c1ecf`                                                                                     |
| `sample.address.update.*.country.PT`                                   | Can update the country of any address to `PT` but not set any other country (unless either a broader permission or permission for another country is also provided) |

Permissions are evaluated for each affected model (and field) when a request is made. If, for example, you send the
payload of an organization with an employee in it both `sample.organization.create` and `sample.employee.create` will be
verified.

We have the internal representations for Users and Roles. Each of these can contain permissions. Authorization is all
based on permissions, not roles. The purpose of the roles is to create groupings of permissions to be able to easily
grant those set of permissions to any given user.

If you wish to use the security and automatic permission validation included in apized you must implement the
UserResolver interface and replace the default implementation (MemoryUserResolver) with your own implementation. Your
UserResolver implementation is then responsible for assembling the Users and Roles for a given caller.

Authentication is assumed to be with tokens. These can be passed via the `Authorization` header with the
format `Bearer [token]` or alternatively via a cookie (particularly useful for web UIs). The cookie name is configurable
with the `apized.cookie` application property and defaults to `token`.

For more advanced use of permissions we also support the concept of inferred permissions. This referes to permissions
the user is granted from the context he is in rather than having the permissions explicitly added to them. This is
particularly useful when dealing with models that have ownership since it allows us to infer the permissions at runtime
instead of adding a huge number of permissions to a user.

Typically, this is done by adding a Filter that can calculate these.

```java

@Filter("/**")
class PermissionContextEnricher implements HttpServerFilter {
  @Inject
  BookingRepository bookingRepository;

  @Inject
  ApizedConfig config;

  @Override
  public Publisher<MutableHttpResponse<?>> doFilter(HttpRequest<?> request, ServerFilterChain chain) {
    User user = ApizedContext.getSecurity().getUser();

    if (ApizedContext.getRequest().getPathVariables().get("booking") != null) {
      Optional<Booking> optBooking = bookingRepository.get(ApizedContext.getRequest().getPathVariables().get("booking"));

      if (optBooking.isPresent() && user.getId().equals(optBooking.get().getOwner())) {
        Booking booking = optBooking.get();
        user.getInferredPermissions().addAll(List.of(
          config.getSlug() + ".booking.get." + booking.getId(),
          config.getSlug() + ".booking.update." + booking.getId() + ".owner"
        ));
      }
    }
    return chain.proceed(request);
  }

  @Override
  public int getOrder() {
    return -1000;
  }
}
```

## Federation

Every instance of every server becomes an API Gateway. The only thing to bear in mind is that you must request the root
object of your query to the servers that contain that model directly.

## Engines

### Micronaut

[Micronaut](https://micronaut.io/) is the default underlying engine for apized. The micronaut
advantages that make it our default choice is the reflection free approach which will drastically improve startup times.
Apized is also fully compatible with native compilation, which means you can deploy light-weight ulta-performant native
binaries.

### Spring Boot

[Spring Boot](https://spring.io/projects/spring-boot) is something we'd like to add in the future given spring boot's
popularity, but we'd need the time and resources to do so. This implementation will probably only happen if this
projects gets wider adoption.
