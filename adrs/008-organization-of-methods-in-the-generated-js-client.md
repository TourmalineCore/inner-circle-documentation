# 008: Organization of Methods in the Generated JS Client: Grouping by Controllers or Flat List

## Status

Accepted (2026-08-25)

## Context
We have a generated JS client that groups methods by controllers.
On the frontend, we need to organize convenient access to these methods.

## Decision
**Grouping by Controllers**
In this case, we use the default organization of API methods without changes. This method does not require additional configuration.
Example of method organization:

```js
{
  tracking: {
    getTaskEntries: () => unknown,
    getUnwellEntries: () => unknown,
  },
  reporting: {
    getReport: () => unknown,
  }
}
```

Example of calling methods:

`api.tracking.getTaskEntries()`

`api.reporting.getReport()`

Example of API client configuration:
```js
const apiClient = new Api({
  baseURL: API_ROOT_URL,
})

initApiInterceptors(apiClient.instance)
```

## Reasoning
- No additional configuration is needed
- No name conflicts
- A flat list does not give any advantages
- Clear separation by features

## Alternatives

## Converting to a Flat List
In this case, we need to add additional configuration to convert the default organization of API methods by controllers into a flat list.

Default organization:
```js
{
  tracking: {
    getTaskEntries: () => unknown,
    getUnwellEntries: () => unknown,
  },
  reporting: {
    getReport: () => unknown,
  }
}
```

Flat list:
```js
{
    getTaskEntries: () => unknown,
    getUnwellEntries: () => unknown,
    getReport: () => unknown,
}
```

In this method, there is also a problem with name conflicts: there may be cases where several controllers have a method with the same name, for example, getList(). To solve this problem, additional OpenAPI configuration on the API side is needed. For example, you can add configuration so that a prefix with the controller name is added to the method name. Then we get the following result:

```js
{
    trackingGetTaskEntries: () => unknown,
    trackingGetUnwellEntries: () => unknown,
    reportingGetReport: () => unknown,
}
```
Comparison of calls
- Flat list: `api.trackingGetTaskEntries()`
- Grouping by controllers: `api.tracking.getTaskEntries()`

As you can see, the flat list does not provide an advantage even in the length of the method name.

## Disadvantages
- Additional configuration is needed
- Complex typing
- Does not give any advantages compared to the chosen method

Example of configuration with typing for converting to a flat list:

```js
const apiClient = new Api({
  baseURL: API_ROOT_URL,
})

initApiInterceptors(apiClient.instance)

// utility type that converts union to intersection
// used to flatten all API controller methods into a single object type
type UnionToIntersection<U> = (U extends any ? (k: U) => void : never) extends (k: infer I) => void ? I : never

// get all methods from API controllers
type ControllerMethods = typeof apiClient[keyof typeof apiClient]

// get final type with all API methods flattened
type CombinedApi = UnionToIntersection<ControllerMethods>

// previously it contained only api object
// now it contains an object per api controller, like tracking, reporting, internal
// to keep things simple for ui we can combine all endpoints into an old single api facade object
const objectsWithEdnpoints = Object.values(apiClient)

const apiWithAllCombinedEndpoints = objectsWithEdnpoints
  .reduce(
    (acc, curr) => ({
      ...acc,
      ...curr, 
    }), 
    {},
  ) as CombinedApi

export const api = apiWithAllCombinedEndpoints
```