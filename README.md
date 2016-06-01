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
