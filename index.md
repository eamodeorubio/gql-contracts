![Cover](./img/MA18_COVERslide_16-9.png) <!-- .element style="max-height:100%"-->


# @eamodeorubio <!-- .element: style="color: yellow;text-transform: none" -->

<a href="https://www.contentful.com/" rel="nofollow" target="_blank"><img src="img/contentful.svg" style="max-width:600px;border:none;background:none;" alt="Powered by Contentful" /></a>


# <span style="color: red">Doomed</span> to <span style="text-transform: none">µService</span>


### Monolith heaven

![Monolith heaven](./img/monolith-heaven.webp)


### Shared ownership?

![Monolith broken](./img/monolith-broken.webp)


## Too many features
### (Anyone heard about SRP?)


### Let's break the monolith
#### (a.k.a <span style="text-transform: none">µService</span>s)

![microservices](./img/microservices-heaven.webp)


### <span style="text-transform: none">µServices</span> === Distributed monolith?

![Distributed Monolith](./img/microservices-broken.gif)


### We need <span class="good">contracts</span> !!!


# Contracts with
# &#x1F680;<span style="color:green;text-transform:none">GraphQL</span>&#x1F680;


### Contracts everywhere

![Contracts everywhere](./img/contracts.jpg)


### What's inside a contract?
#### <span style="color:yellow">Data model</span>

```protobuf
message PersonalInfo {
  required string name;
  optional string email;
}

message Employee {
  required number employeeId;
  required PersonalInfo info;
}
```


### What's inside a contract?
#### <span style="color:yellow">Data over the network</span>

```json
{
  "employeeId": 33,
  "info": {
    "name": "John",
    "email": null
  }
}
```


### What's inside a contract?
#### <span style="color:yellow">Operations</span>

```protobuf
service EmployeeService {
  rpc hire(PersonalInfo) returns (Employee);
}
```


### What's inside a contract?
#### <span style="color:yellow">Operations over the network</span>

```http
POST /employee/rpc HTTP/1.1
Host: example.org
Authorization: Bearer AbCdEf123456
Content-Type: application/json; charset=utf-8

{
  "op": "hire",
  "args": [{
    "name": "John",
    "email": null
  }]
}
```


### What's inside a contract?
#### <span style="color:yellow">Operations over the network</span>

```http
HTTP/1.1 200 OK
Date: Sun, 10 Oct 2010 23:26:07 GMT
Server: Apache/2.2.8 (Ubuntu) mod_ssl/2.2.8 OpenSSL/0.9.8g
Last-Modified: Sun, 26 Sep 2010 22:04:35 GMT
ETag: "45b6-834-49130cc1182c0"
Content-Type: application/json; charset=utf-8

{
  "employeeId": 33,
  "info": {
    "name": "John",
    "email": null
  }
}
```


### What's inside a contract?
#### <span style="color:yellow">Input/Output invariants</span>

* `findById(id: string): Employee` -> If found, we expect the result to have the same `id` as specified
* `updateName(id: string, name: string): Employee` -> If successful, we expect the returned employee to have the supplied name


## So what has <span class="good" style="text-transform:none">GraphQL</span> to do with contracts?


## <span class="good" style="text-transform:none">GraphQL</span> does define a contract!


## GraphQL does *not* mandate how to serialise data


### But gives a JSON serialisation Recommendation

https://facebook.github.io/graphql/June2018/#sec-Serialization-Format


## GraphQL does *not* define how to transport it over the network


### But there is a <span style="color:yellow;font-style:italic">de facto</span> standard

* Single endpoint: `/graphql`
* Use `GET` and `POST` ignoring their semantics
* Use `application/json` and JSON serialisation
* `4xx` for validation and security errors
* `200` for anything else (partial success)
* https://graphql.org/learn/serving-over-http/


### SDL
#### Data models

```graphql
scalar ProductRef
scalar Rating
enum ReviewStatus {
  PENDING
  ACCEPTED
  REJECTED
}
type ProductReview {
  id: ID!
  productRef: ProductRef!
  rating: Rating!
  commentary: String
  status: ReviewStatus!
}
```


### SDL
#### Queries &amp; Mutations

```graphql
input ReviewData {
  productRef: ProductId!
  rating: Rating!
  commentary: String
}

type Query {
  productReview(id: ReviewId!): ProductReview
}

type Mutation {
  postReview(review: ReviewData!): ProductReview!
  deleteReview(id: ReviewId!): boolean
}
```


### Query language

```graphql
query {
  review1: productReview(id: "1") {
    ...interestingFields
  }
  review2: productReview(id: "2") {
    ...interestingFields
  }
  review3: productReview(id: "3") {
    ...interestingFields
  }
}

fragment interestingFields on ProductReview {
  productRef
  rating
}
```


### Errors &amp; partial failures

```json
{
  "data": {
    "review1": null,
    "review2": {
      "productRef": "urn:ishop:product:2",
      "rating": 4
    },
    "review3": null
  },
  "errors": [
    {
      "message": "Unexpected DB Error",
      "locations": [ { "line": 2, "column": 3 } ],
      "path": [ "review1" ]
    }
  ]
}
```


### <span style="color:green;text-transform:none">GraphQL</span> + Conventions = Contract?

* SDL
* [GraphQL spec](https://facebook.github.io/graphql/June2018/) for extra details like errors
* De facto standard: [GraphQL over HTTP](https://graphql.org/learn/serving-over-http/)
* De facto standard: [GraphQL serialised as JSON](https://facebook.github.io/graphql/June2018/#sec-Serialization-Format)


# Testing <span style="color:green;text-transform:none">GraphQL</span> Contracts


## We should test ... What exactly?


### <span class="bad">Not</span> a contract test

* Correct implementation of business logic
  * In the server
  * In the clients
* The full end to end behaviour


### Both sides adhere to the contract

* Valid means it complies with the contract
* Clients:
  * Perform <span class="good">valid requests</span>
  * Can process any <span class="good">valid results</span>
* Servers:
  * Accept <span class="good">valid requests</span>
  * Always return a <span class="good">valid result</span>


### Testing the client side

![Consumer Contract Tests](./img/consumer-contract-test.jpg)


### Testing the server side

![Provider Contract Tests](./img/provider-contract-test.jpg)


### Problems

* Tons of boilerplate
* Prone to have bugs
* Test needs to be kept in sync
* Not <em><span class="bad">DRY</span></em> !!


### <span class="good" style="text-transform:none">GraphQL</span> to the rescue!

* Strong Typed SDL give us automation
  * Test data generators
  * Mock Servers
  * Test scenarios scaffolding
* No need to test type validation on the server


### Consider this SDL

```gql
scalar ProductRef
scalar Rating
enum ReviewStatus {
  PENDING
  ACCEPTED
  REJECTED
}
type ProductReview {
  id: ID!
  productRef: ProductRef!
  rating: Rating!
  commentary: String
  status: ReviewStatus!
}
```


### Generating Types &amp; scaffolding

[For example with `GraphQLGen`](https://github.com/prisma/graphqlgen)

```typescript
type ID = string
type ProductRef = string
type Rating = number
enum ReviewStatus {
  'PENDING',
  'ACCEPTED',
  'REJECTED'
}
type ProductReview = {
  id: ID
  productRef: ProductRef
  rating: Rating
  commentary: string | null
  status: ReviewStatus
}
```


### Fake Data for tests
#### A maintenance burden

```json
[
  {
    "id": "2f4dc6ba-bd25-4e66-b369-43a13e0cf150",
    "productRef": "urn:ishop:product:2c497439-7802-4cd5-8fb6-f7fa679cef7d",
    "rating": 4,
    "commentary": null,
    "status": "PENDING"
  },
  {
    "id": "a9af522d-0d62-46f3-8c1d-71eff807fcc7",
    "productRef": "urn:ishop:product:bb509f54-e20f-4fbb-8df7-adeb7b83cf4b",
    "rating": 2,
    "commentary": "Vel et rerum nostrum quia. Dolorum fuga nobis sit natus consequatur.",
    "status": "ACCEPTED"
  }
]
```


### Fake Data Generators
#### Custom Scalars

```typescript
import * as casual from 'casual'

const someID = (): ID => casual.uuid

const someProductRef = (): ProductRef =>
  `urn:ishop:product:${someID()}`

const someRating = (): Rating => casual.integer(1, 5)
```


### Fake Data Generator
#### Enums &amp; Strings

```typescript
import * as casual from 'casual'

const someReviewStatus =
  () => casual.random_value(ReviewStatus)

type StringTypes =
  'title' | 'text' | 'description' | 'string'

const someString = (
  { kind = 'string' }: { kind?: StringTypes } = {}
): string => casual[kind]
```


### Fake Data Generator
#### Object Types

```typescript
import * as casual from 'casual'

const someProductReview =
  (options: Partial<ProductReview> = {}) => ({
    id: someID(),
    productRef: someProductRef(),
    rating: someRating(),
    commentary: casual.boolean ?
      someString({ kind: 'description'}) :
      null,
    status: someReviewStatus(),
    ...options
  })
```


### Fake Data Generator Generators
#### Removing boilerplate

* Most of the code of the generators are highly regular
* Only custom scalars need to be defined
* And some minor customisations


### Fake Data Generator Generators
#### Removing boilerplate

```typescript
// Not (yet) a real library
import { load, nullable, someString } from 'gql-faker'
const myFaker = load('../schemas/reviews.graphql')
myFaker.customScalars({
  someID: (): ID => casual.uuid,
  someProductRef: (): ProductRef =>
    `urn:ishop:product:${myFaker.someID()}`,
  someRating: (): Rating => casual.integer(0, 5)
})
myFaker.customiseObject('ProductReview', {
  commentary: nullable(
    () => someString({
      kind: 'description'
    })
  )
})
```


### Consider these Operations

```graphql
input ReviewData {
  productRef: ProductId!
  rating: Rating!
  commentary: String
}

type Query {
  productReview(id: ReviewId!): ProductReview
}

type Mutation {
  postReview(review: ReviewData!): ProductReview!
  deleteReview(id: ReviewId!): boolean
}
```


#### Test Scenarios Generator

```js
myFaker.testScenarios({
  Query: {
    // productReview(id: ReviewId!): ProductReview
    productReview: [
      {
        name: 'found',
        result: (id) => someProductReview({ id })
      },
      {
        name: 'not found',
        result: null
      }
    ]
  }
})
```


#### Test Scenarios Definition

```js
myFaker.testScenarios({
  Mutation: {
    // postReview(review: ReviewData!): ProductReview!
    postReview: [
      {
        name: 'with a commentary',
        input: () => someReviewData({ commentary: someString() }),
        result: (data) => someProductReview({ ...data })
      },
      {
        name: 'without a commentary',
        input: () => someReviewData({ commentary: null }),
        result: (data) => someProductReview({ ...data })
      },
      {
        name: 'failure',
        result: (data) => alreadyExistsError(data.id)
      }
    ]
  }
})
```


### Test Scenarios Definition
#### What about `deleteReview`?

```js
// No invariants
// Return value is just a boolean
// Test scenario can be automatically inferred
myFaker.testScenarios({
  Mutation: {
    // deleteReview(id: ReviewId!): boolean
    deleteReview: [
      { name: 'returns true', result: true },
      { name: 'returns false', result: false },
      { name: 'failure', result: () => someSystemError() }
    ]
  }
})
```


### No boilerplate
#### Only need to define what is <em>not</em> in the SDL

* Custom Scalars
* Domain customisation for Objects and Inputs
* Invariants inputs/outputs
* Errors (not in SDL)
* Edge cases


### Automate!!!

![Tooling](./img/tools.jpg)


### Automate!!!

![Consumer Contract Tests Tools](./img/consumer-contract-test-auto.jpg)


### Automate!!!

![Provider Contract Tests](./img/provider-contract-test-auto.jpg)


# The <span style="color:green">Evergreen</span> Contract


### Contract Drift Problem

1. Provider change contract
2. Consumer is not aware of the change
3. Mock server is not updated
4. False passing tests
5. Bug!


### Use <span style="color:green">evergreen</span> contracts to avoid <span class="bad">Contract Drift</span>


### Don't <span class="bad">SEMVER</span>

* Only contract version in <em>production</em> is relevant
* Fix clients ASAP
* Save money, don't maintain several versions at the same time


### <span class="good">Evergreen</span>

* Deprecate and notify
* Measure usage of deprecated operations &amp; types
* Drop them when usage is low enough
* Contract changed? -> <em>Run contract tests for <strong>all</strong> services consuming that contract</em>


### Need <em>Service Directory</em> to know for which services their contract tests need to be run


### Service Directory

* Unique source of truth regarding Contracts
* Keeps tracks of:
  * Which services provide which Contracts
  * Which services consume which contracts
  * Contracts = SDL + Test Scenario Definitions
  * Well known test cases


### Service Directory
#### Keeping it up to date

* Each service repo contains a descriptor
  * Provided contracts definitions
  * Reference to all consumed contracts
* On commit `push` -> push descriptor to service directory


### Service Directory
#### Humans!

* Search & consult the contracts and their test cases
* Report service dependencies
* Report services with broken contract tests (provider &amp; consumer)
* Report services using deprecated contract features


### Service Directory
#### For testing &amp; CI

* Tests ask service directory to spawn a:
  * Mock server
  * Automated consumer contract test
* CI notify directory of broken contract tests
* CI notify directory of deprecated contract usage


### Service Directory
#### Enforcing <span class="good">Evergreen</span> contracts

* On contract change -> Trigger contract tests for any service consuming that contract
* In production, use telemetry to report any deprecated usage or broken calls to the directory
