# Testing methodology

## Testing the deployment time

To ensure that deploy is not affected by internet speed, the `pullPolicy: IfNotPresent` configuration will be set for all services. This means that images will be downloaded from the internet once, and the already downloaded images will be used thereafter. Next, we create a cluster and deploy the services to the `local` namespace. This way, we download the images, and because of the `pullPolicy`, they will not be downloaded again. Next, we delete the namespace to remove all services and their configurations, and then create the namespace again to avoid spending time on its creation. From this point, the cluster is ready for testing.

To measure the deployment time, we’ll run `helmfile apply` using the `time` utility. Important: We’ll run `helmfile apply` without `cache cleanup` so the charts downloaded from the internet aren’t cleared.

Each measurement will be taken three times, after which the average value will be calculated. All measurements will be taken without CPU usage limits, because situations where containers use a lot of CPU do not cause the container to crash.

### Commands to Run

Deleting a namespace
```bash
kubectl delete namespace local
```

Creating a namespace
```bash
kubectl create namespace local
```

Deploying services
```bash
time helmfile --environment local --namespace local -f deploy/helmfile.yaml apply
```


### Testing with minimum requests and “maximum” limits

Limits: 10000m 1000Mi
Requests: 1m 1Mi

114 sec
118 sec
115 sec

avg: 115 sec

### Testing with high requests and “maximum” limits

Limits: 10000 m 1000 Mi
Requests: 500 m 500 Mi

116 sec
119 sec
119 sec

avg: 118 sec

### Testing with minimum requests and a 300Mi limit

Limits: 10000m 300Mi
Requests: 1m 1Mi

113 sec
116 sec
121 sec

avg: 116 sec

### Testing with minimum requests and a 200Mi limit

Limits: 10000m 200Mi
Requests: 1m 1Mi

117 sec
124 sec
112 sec

avg: 117 sec

### Testing with minimum requests and a 100Mi limit

Limits: 10000m 100Mi
Requests: 1m 1Mi

117 sec
118 sec
116 sec

avg: 117 sec

## Testing with minimum requests and a 90Mi limit

Limits: 10000m 90Mi
Requests: 1m 1Mi

Failed to deploy

### Conclusion on the impact of memory on deployment time

Minimum and maximum requests did not affect deployment time. Only low limits affected deployment. N ot the time, but the ability to deploy services.

Next, we’ll manually test all functionality via the UI to obtain the values we expect to see when using IC locally. We’ll then set the request limits based on these values.

## Manual testing

### API

auth-api: 177Mi
accounts-api: 174Mi
books-api: 139Mi
compensations-api: 112Mi
documents-api (no test document—could not be tested): 67Mi
email-sender (we do not test the documents-api—could not be tested): 45Mi
employees-api: 191Mi
invoices-api: 49Mi
items-api(отстутствует UI - не удалось протестировать): 55Mi
time-api: 101Mi

### UI
```
auth-ui: 13Mi
accounts-ui: 13Mi
books-ui: 13Mi
compensations-ui: 13Mi
documents-ui: 13Mi
invoices-ui: 13Mi
layout-ui: 13Mi
time-ui: 13Mi
inner-circle-ui: 14Mi
```

### Manual testing conclusion

#### UI

All services have an average usage of 13Mi, so we’ll set the request limit to 25Mi for all of them. We’ll set the overall limit with a generous margin 75Mi.

#### API

Each service has different usage patterns, so the request limits will vary for each one.

## K6

All test scenarios could be found in the [inner-circle-load-testing-experimental](https://github.com/TourmalineCore/inner-circle-load-testing-experimental) repo.

Results:

```
auth-api: 297Mi
accounts-api: 333Mi
books-api: 287Mi
compensations-api: 251Mi
employees-api: 333Mi
invoices-api: 62Mi
time-api: 117Mi
items-api: 115Mi
documents-api and email-sender: Like the invoices-api, these do not interact with the database, so we expect similar consumption and will set them to 62Mi
```

We will set the limits to the maximum consumption + 25% (rounding up is allowed)
```
auth-api: 375Mi
accounts-api: 425Mi
books-api: 375Mi
compensations-api: 325Mi
employees-api: 425Mi
invoices-api: 100Mi
time-api: 150Mi
items-api: 150Mi
documents-api and email-sender: 100Mi


Without any changes
```
  Resource           Requests      Limits
  --------           --------      ------
  memory             2826Mi (20%)  5940Mi (42%)
```
```
Applied new UI requests and limits
  Resource           Requests      Limits
  --------           --------      ------
  memory             2601Mi (18%)  3915Mi (28%)
```
```
Applied new UI and API requests and limits
  Resource           Requests      Limits
  --------           --------      ------
  memory             2111Mi (15%)  3465Mi (25%)
```