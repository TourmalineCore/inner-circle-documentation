# 010: Reusable helper in Karate tests

## Status

Proposed (2026-10-27)

This document describes the research and two helper options that have been implemented. Each option is used in one e2e scenario to compare the approaches. Choosing one option and updating the other places that create Task Entries are part of the next stage of the task.

## Context

The logic for creating a Task Entry was repeated 7 times in 5 feature files. Each place had the same steps:

- Get the list of projects for the entry date and choose a `projectId`
- Create a Task Entry with `POST tracking/task-entries`
- Check for HTTP status `200`
- Get the ID from `response.newTaskEntryId`

Goal: move entry creation to a shared helper and keep the authorization set up by the calling test. This is important because a test can switch between users in one scenario.

The task requirements for the shared helper are:

- Require `startTime`, `endTime`, and a `projectId` passed directly to the helper
- Accept optional `title`, `taskId`, and `description`
- Use default values when optional parameters are not provided
- Return the created entry with its ID

Two options are described below. In both options, a separate helper gets the projects, and the required `projectId` is passed directly to the creation helper.

## Options considered

### Karate feature files

Entry creation is implemented in `e2e/helpers/create-task-entry.feature`.

The helper checks that `startTime`, `endTime`, and `projectId` are present before sending the request.

Main logic:

```gherkin
@ignore
Feature: Create Task Entry (helper, not a test)

  Scenario:
    * def args = karate.get('__arg', {})
    * if (!args.startTime || !args.endTime) karate.fail('createTaskEntry: startTime and endTime are required')
    * if (args.projectId == null) karate.fail('createTaskEntry: projectId is required')

    * def taskEntry =
    """
    ({
      title: args.title || '[API-E2E]-Test-task-entry-' + Math.random(),
      startTime: args.startTime,
      endTime: args.endTime,
      projectId: args.projectId,
      taskId: args.taskId || '#2233',
      description: args.description || 'Task description'
    })
    """

    Given url apiRootUrl
    And path 'tracking/task-entries'
    And request taskEntry
    When method POST
    Then status 200

    * taskEntry.id = response.newTaskEntryId
```

A separate file, `e2e/helpers/get-projects.feature`, gets the list of projects. It accepts `startDate` and `endDate` through `__arg` and saves the list from the response in the `projects` variable. The calling scenario chooses a project from the list:

```gherkin
@ignore
Feature: Get Projects

    Scenario:
        * def startDate = __arg.startDate
        * def endDate = __arg.endDate

        Given url apiRootUrl
        And path 'tracking/task-entries/projects'
        And params { startDate: '#(startDate)', endDate: '#(endDate)' }
        When method GET
        Then status 200

        * def projects = response.projects
```
`@ignore` prevents the helper feature from running as a separate test.

Example of calling the helpers after setting up authorization:

```gherkin
* def getProjectsResult = call read('helpers/get-projects.feature') { startDate: '2030-11-05', endDate: '2030-11-05' }
* def projectsList = getProjectsResult.projects

* def createTaskEntryResult = call read('helpers/create-task-entry.feature') { startTime: '2030-11-05T14:00:00', endTime: '2030-11-05T16:00:00', projectId: '#(projectsList[0].id)' }

* def newTaskEntryId = createTaskEntryResult.taskEntry.id
```

The call results are saved in `getProjectsResult` and `createTaskEntryResult`. The project list comes from `getProjectsResult.projects`, and the created entry comes from `createTaskEntryResult.taskEntry`. The entry contains the data sent in the request and the `id` field from `response.newTaskEntryId`. Saving the result in a variable keeps the helper variables inside a separate object, so they do not overwrite scenario variables with the same names.

**Advantages**

- Uses headers from `configure headers`: there is no need to pass them in each call
- HTTP requests and checks appear as separate steps in the Karate report

**Disadvantages**

- To keep helper variables separate, the result of `call` must be saved in a separate variable. A call without this assignment can overwrite scenario variables with the same names

**Usage details**

- `call` returns an object with the feature file variables. The created entry is available as `createTaskEntryResult.taskEntry`
- The helper reads the passed parameters through `__arg` to clearly separate them from the calling scenario variables

### JavaScript functions in `js-utils.js`

The functions `getProjects(headers, args)` and `createTaskEntry(headers, args)` are implemented in `e2e/js-utils.js`. They send HTTP requests through `karate.http`, without separate Karate feature files.

The API URL is read from the `API_ROOT_URL` environment variable. Both functions take headers as the first argument and parameters as the second. `getProjects` accepts `startDate` and `endDate`, checks for HTTP status `200`, and returns the `response.body.projects` array. The calling scenario chooses a project from the array and passes its ID to `createTaskEntry`. The creation helper checks that `startTime`, `endTime`, and `projectId` are present before sending the POST request.

**Implementation in `js-utils.js`**

```javascript
    getProjects: function (headers, args) {
      var apiRootUrl = this.getEnvVariable('API_ROOT_URL');

      var response = withHeaders(karate.http(apiRootUrl)
        .path('tracking/task-entries/projects'), headers)
        .param('startDate', args.startDate)
        .param('endDate', args.endDate)
        .get();

      if (response.status != 200) {
        karate.fail('getProjects: expected 200 but got ' + response.status);
      }

      return response.body.projects;
    },

    createTaskEntry: function (headers, args) {
      if (!args || !args.startTime || !args.endTime) {
        karate.fail('createTaskEntry: startTime and endTime are required');
      }

      if (args.projectId == null) {
        karate.fail('createTaskEntry: projectId is required');
      }

      var apiRootUrl = this.getEnvVariable('API_ROOT_URL');

      var taskEntry = {
        title: args.title || '[API-E2E]-Test-task-entry-' + Math.random(),
        startTime: args.startTime,
        endTime: args.endTime,
        projectId: args.projectId,
        taskId: args.taskId || '#2233',
        description: args.description || 'Task description',
      };

      var response = withHeaders(karate.http(apiRootUrl)
        .path('tracking/task-entries'), headers)
        .post(taskEntry);

      if (response.status != 200) {
        karate.fail('createTaskEntry: expected 200 but got ' + response.status);
      }

      taskEntry.id = response.body.newTaskEntryId;

      return taskEntry;
    },

  function withHeaders(request, headers) {
    // karate.http ignores `configure headers` and its .headers(map) is not callable from JS,
    // so the caller's headers are applied one by one.
    // karate.toJson turns a Java Map (e.g. jsUtils().getAuthHeaders() passed inline) into a plain object
    var plainHeaders = karate.toJson(headers);
    Object.keys(plainHeaders).forEach(function (key) {
      request = request.header(key, plainHeaders[key]);
    });
    return request;
  }
```

The shared function `withHeaders(request, headers)` adds headers to GET and POST requests. First, `karate.toJson(headers)` converts them from a Java Map to a JavaScript object. Then the function adds each header to the request with `.header(name, value)`.

**Call example**

After authorization and getting an `accessToken`, the scenario gets the projects and creates an entry in separate steps:

```gherkin
* def accessToken = karate.toMap(response.accessToken.value)

* def projectsList = jsUtils().getProjects(jsUtils().getAuthHeaders(accessToken), { startDate: '2033-11-05', endDate: '2033-11-05' })

* def taskEntry = jsUtils().createTaskEntry(jsUtils().getAuthHeaders(accessToken), { startTime: '2033-11-05T14:00:00', endTime: '2033-11-05T16:00:00', projectId: projectsList[0].id })

* def newTaskEntryId = taskEntry.id
```
Headers are passed directly to `getProjects()` and `createTaskEntry()` because `karate.http` does not use the calling scenario's `configure headers` settings. This also allows the scenario to pass the current headers when switching users.

**Advantages**

- The function returns the entry directly: the result is available as `taskEntry`, without the extra object in `createTaskEntryResult.taskEntry`
- Local function variables are not added to the calling scenario's context

**Disadvantages**

- `karate.http` does not use headers from `configure headers`, so they must be passed directly
- A separate HTTP status check must be written
- HTTP requests run inside JavaScript, rather than as separate Karate steps
