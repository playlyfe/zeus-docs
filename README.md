# Basic Introduction

This repository is intended to be a simple introduction to showcasing some of the core API capabilities of the new 
Playlyfe Engine. It does not contain an exhaustive documentation of the API currently. Since graphql is very intuitive in
nature we hope that most developers can use this to starting point and the developer console to discover the learn what
kind of data can be queried by the API.

For an introduction to GraphQL go [here](https://facebook.github.io/react/blog/2015/05/01/graphql-introduction.html)

You can simply copy paste the examples in this file into the developer console to try it out and play with the API.

## Create a Game

```graphql
mutation GameCreate {
  GameCreate(input: {sid: "test", name: "Test", description: "This is a test game !!!", clientMutationId: "1", timezone: "Asia/Kolkata"}) {
    clientMutationId
    game {
      id
      name
      image
      timezone
      meta
    }
  }
}
```

## Create a New Design
A design is a set of gamification components. You can have multiple design versions within a single game.

```graphql
mutation DesignCreate {
  DesignCreate(input: {clientMutationId: "1", sid: "latest", name: "Latest", gameID: "test"}) {
    clientMutationId
    design {
      id
      name
    }
  }
}
```

## Create a New Runtime
A runtime is an environment consisting of a several players into which the design can be deployed. 
A game can contain multiple runtimes. Each runtime can only deploy a single design within it.
You can create different runtimes for staging, test, qa, etc to enable you to easily test the same design in different
environments.

```graphql
mutation RuntimeCreate {
  RuntimeCreate(input: {clientMutationId: "1", sid: "prod", name: "Production", gameID: "test"}) {
    clientMutationId
    runtime {
      id
      name
    }
  }
}
```

## Create a New Metric
You may notice the metric design is serialized json. This is due to a limitation within graphql that makes it difficult
to work with nested, recursive json designs. This limitation can be overcome by using intelligent client SDKs which
automatically do the necessary serialization for fields that require it.

```graphql
mutation MetricCreate {
  DesignComponentCreate(input: {sid: "experience5", name: "Experience", type: metric, versionID: "latest", gameID: "test", clientMutationId: "1", design: "{\"type\":\"point\",\"constraints\":{\"aggregationFunction\":\"sum\",\"better\":\"higher\"}}"}) {
    clientMutationId
    component {
      __typename
      ... on MetricComponent {
        id
        name
        description
        meta
        type
        design {
          type
          constraints {
            ... on PointMetricConstraints {
              aggregationFunction
              better
            }
          }
        }
      }
    }
  }
}
```

## Create a New Action

You will notice in the mutation below we perform two operations: `DesignComponentCreate` and `DesignComponentUpdate`.
You can easily perform multiple mutations together within a single request with graphql. This is perfect when you 
are looking to optimize network round trips in your application. Eg: Batch creation of large number of players

Another advantage of batching mutations is that they are treated as a single atomic unit within a transaction.
If any of the mutations fail then all of them will be rollbacked. This is useful when you need to maintain data
integrity between the gamification engine and your application.

```graphql
mutation ActionCreate {
  DesignComponentCreate(input: {sid: "login", name: "Login", type: action, versionID: "latest", gameID: "test", clientMutationId: "1", design: "{\"type\":\"point\",\"constraints\":{\"aggregationFunction\":\"sum\",\"better\":\"higher\"}}"}) {
    clientMutationId
    component {
      __typename
      ... on MetricComponent {
        id
        name
        description
        meta
        type
        design {
          type
          constraints {
            ... on PointMetricConstraints {
              aggregationFunction
              better
            }
          }
        }
      }
    }
  }
  DesignComponentUpdate(input: {id: "{\"gameID\":\"test\",\"id\":\"login\",\"type\":\"ActionComponent\",\"versionID\":\"latest\"}", name: "Login 4", meta: "{ \"test\": true }", clientMutationId: "1", design: "{\"enabled\":true,\"execute\":{\"type\":\"program\",\"params\":{\"body\":[{\"type\":\"reward\",\"params\":{\"metric\":{\"id\":\"experience5\"},\"target\":\"event.actor\",\"operator\":\"add\",\"value\":5}}]}}}"}) {
    clientMutationId
    component {
      __typename
      ... on ActionComponent {
        id
        name
        description
        meta
        type
        design {
          enabled
          execute
          variables {
            ... on BooleanVariable {
              id
              name
              required
              default
              type
            }
            ... on StringVariable {
              id
              name
              required
              default
              type
            }
            ... on NumberVariable {
              id
              name
              required
              default
              type
            }
          }
        }
      }
    }
  }
}
```

## Deploy 

```graphql
mutation Deploy {
  Deploy(input: {clientMutationId: "1", runtimeID: "{\"gameID\":\"test\",\"id\":\"prod\",\"type\":\"Runtime\"}", designID: "{\"gameID\":\"test\",\"id\":\"latest\",\"type\":\"Version\"}"}) {
    clientMutationId
    runtime {
      id
      sid
      name
    }
  }
}
```

## Player Create

```graphql
mutation PlayerCreate {
  PlayerCreate(input: {clientMutationId: "x", sid: "tester1", name: "Tester 1", gameID: "test", runtimeID: "prod"}) {
    clientMutationId
    player {
      id
      sid
      name
      teams {
        id
        name
      }
    }
  }
}
```

## Player Action Play

This is perhaps one of the most commonly used mutation in a gamification system. As you can see the new graphql API
enables you to perform the mutation and fetch all releavant information like the activity information, player 
profile, etc with a single request. This can again be a big performance boost over slow networks.

```graphql
mutation PlayerActionPlay {
  PlayerActionPlay(input: {clientMutationId: "y", playerID: "{\"gameID\":\"test\",\"id\":\"tester1\",\"runtimeID\":\"prod\",\"type\":\"Player\"}", actionID: "{\"gameID\":\"test\",\"id\":\"login\",\"runtimeID\":\"prod\",\"type\":\"ActionRuntimeComponent\"}", variables: "{}"}) {
    clientMutationId
    activity {
      id
      timestamp
      changes {
        type
        context {
          id
          type
        }
        data
      }
      data
      output
    }
    player {
      id
      sid
      scores {
        ... on PointScore {
          metric {
            name
          }
          value
        }
        ... on SetScore {
          metric {
            name
          }
          value {
            id
            name
            description
            value
          }
        }
        ... on StateScore {
          id
          metric {
            name
          }
          description
          value
        }
      }
      teams {
        id
        name
        roles {
          id
        }
      }
    }
  }
}
```

# Advanced Concepts

There are some basic concepts that must be understood while working with the API. The Playlyfe GraphQL API is compatible 
with the [Relay Specification](https://facebook.github.io/relay/docs/graphql-relay-specification.html#content). Relay is a framework convention over the basic GraphQL specification that helps solve common problems like pagination, data mutation and data caching for clients. We decided to conform to this specification since to effectively solves many common scenarios and also ensures wider compatibility with the GraphQL-Relay eco-system that is growing. It is recommended to go through the relay specification to better understand the convention.

Relay requires every object within the system to be uniquely identifiable with a Global ID. This Global ID is used when fetching data through connections and also as cache keys. We do have some noticable additions/difference from relay due to the nature of our system which I will explain below.

## Global ID Structure

Every object within the system has a unique Global ID which is represented as a JSON object(this is returned as the `id` key on the object, not to be confused with `sid`). The relay specification recommends base64 encoding global IDs to make them opaque and prevent clients from writing hacky code that messes with whatever convention the backend may follow for global IDs. 
However this proves to be more of a limitation than a simplification in backend integration scenarios where the client often has a lot more context to be able to select specific objects. As a result we decided to make global IDs a transparent easily parseable and generatable object. The structure of a Global ID is as follows:

- It is a json object
- It contains a `type` key which specifies the type of the object
- It contains an `id` key which corresponds to the `sid` of the object
- It contains one or more other keys which specifies other parameters which uniquely identifies the object. Eg: A player is specified by `gameID`, `runtimeID` and a `id`. A runtime is specified via `gameID` and `id`. 

## SIDs vs GlobalIDs

**SID** stand for **Short ID**. Often when we are working with mutations we would like to create objects using simple Short IDs as Global IDs can be long and messy. However when we are updating or deleting objects it is simpler to pass a single GlobalID to identify the object that is being updated/deleted. As a result we use the following convention within our API.

- All object creation is done using SID. Once the object is created you can fetch the Global ID of the object from the Mutation Payload.
- All update and delete mutations select the object they operate on using Global IDs.








