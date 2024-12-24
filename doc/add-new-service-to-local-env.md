## Чтобы добавить новый сервис в local-env нам нужно:

### В репозиториии local-env:

- добавить информацию о новом сервисе в `helmfile.yaml` в папке `deploy`

```
- name: # your-service-name from service package.json
   labels:
     app:  # your-service-name from service package.json
   wait: true
   chart: bitnami/nginx #you need vpn for it   
   # after 15.3.5 our docker file or setup can no longer start, need to investigate what is wrong for the newer versions
   version: 15.3.5
   # it won't work anyway until ingress controller is created
   # thus we wait for it to be ready first
   needs:
     - ingress-nginx
   values:
     # https://helmfile.readthedocs.io/en/latest/#loading-remote-environment-values-files
     - git::https://github.com/<your-org-name>/<your-service-name>.git@/ci/values-local-env.yaml?ref=
<your-branch-name> (ref=master or ref=feature/<your-brunch-name>)
     - values.yaml.gotmpl
```

Пример:

```
- name: inner-layout-ui
    labels:
      app: inner-layout-ui
    wait: true
    chart: bitnami/nginx
    # after 15.3.5 our docker file or setup can no longer start, need to investigate what is wrong for the newer versions
    version: 15.3.5
    # it won't work anyway until ingress controller is created
    # thus we wait for it to be ready first
    needs: 
      - ingress-nginx
    values:
      # https://helmfile.readthedocs.io/en/latest/#loading-remote-environment-values-files
      - git::https://github.com/TourmalineCore/inner-circle-layout-ui.git@/ci/values-local-env.yaml?ref=feature/replace-layout
      - values.yaml.gotmpl
```

### В сервисе, который подключаем в local-env (в нашем случае [inner-circle-layout-ui](https://github.com/TourmalineCore/inner-circle-layout-ui))

- создайте или обновите `Dockerfile`, он понадобится для сборки образа и дальнейшего его использования в local-env

```
FROM node:20.11.1-alpine3.19 as build
COPY package.json .
COPY package-lock.json .
RUN npm ci
COPY . .
RUN npm run build -- --base=/layout

FROM nginx:1.26.0-alpine3.19-slim
COPY /ci/nginx.conf /data/conf/nginx.conf
COPY --from=build /dist /usr/share/nginx/html

EXPOSE 80

WORKDIR /usr/share/nginx/html
RUN apk add --no-cache bash

CMD nginx -g "daemon off;" -c "/data/conf/nginx.conf"
```

- создайте папку `.github/workflows` и в нем файл `docker-build-and-push.yml` для сборки образа и его публикации в GitHub Container Registry

```
name: Publish Docker image

on:
 push:
   branches:
     - master
     - feature/*
     
jobs:
 push_to_registry:
   name: Push Docker image to GitHub
   runs-on: ubuntu-22.04
   permissions:
     packages: write
     contents: read
     attestations: write
   steps:
     - name: Check out the repo
       uses: actions/checkout@v4

     - name: Log in to GitHub Container Registry
       uses: docker/login-action@v2
       with:
         registry: ghcr.io
         username: ${{ github.actor }}
         password: ${{ secrets.GITHUB_TOKEN }}

     - name: Build and push Docker image
       uses: docker/build-push-action@v5
       with:
         context: .
         file: ./Dockerfile
         push: true
         tags: ghcr.io/<your-org-name>/<your-app-name>/
         <your-service-name>:${{ github.sha }} 
         # for example: tags: ghcr.io/tourmalinecore/inner-circle/layout-ui:${{ github.sha }}
```

- также в корне проекта нужно создать папку `ci`, если у вас ее еще нет и добавить туда `helmfile.yaml`

```
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami

releases:
  - name: layout-ui
    labels:
      app: layout-ui
    wait: true
    chart: bitnami/nginx
    # after 15.3.5 our docker file or setup can no longer start, need to investigate what is wrong for the newer versions
    version: 15.3.5
    values:
      - values.yaml
```

- в папке `ci` создаем файл `nginx.conf` для настройки `nginx`

```
worker_processes auto;

events {
    worker_connections  1024;
    multi_accept on;
}

http {
    server {
        listen 80;
        server_name  localhost;

        root   /usr/share/nginx/html;
        index  index.html index.htm;
        include /etc/nginx/mime.types;

        gzip on;
        gzip_min_length 1000;
        gzip_proxied expired no-cache no-store private auth;
        gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

        location / {
            add_header Cache-Control 'no-cache';
            try_files $uri $uri/ /index.html;
        }
    }
}
```

- в той же папке добавляем файл `values-local-env.yaml` где описываем настройки параметров Helm-чарта добавляемого сервиса

```
# Here you can find docs bitnami/nginx chart https://github.com/bitnami/charts/blob/main/bitnami/nginx/README.md

image:
  registry: ghcr.io
  repository: tourmalinecore/inner-circle/layout-ui
  tag: "latest"
  pullPolicy: Always
  debug: true

containerPorts:
  http: 80
  https: ""

replicaCount: 1

# 1000m means 100% of processor time 
# 1m means 0.1% of processor time.
# at start pod is being allocated with resources from requests, if it needs more consumption can grow until limits.
# if pod uses more resources than limits in spite of the reason kube-system will perform a forced restart
resources:
  limits:
    cpu: 50m
    memory: 100Mi
  requests:
    cpu: 1m
    memory: 50Mi

livenessProbe:
  enabled: false

readinessProbe:
  enabled: false

service:
  type: ClusterIP
  ports:
    http: 80
    https: ""

ingress:
  enabled: true
  path: /layout(/|$)(.*)
  annotations:
      cert-manager.io/cluster-issuer: letsencrypt 
      nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
      nginx.ingress.kubernetes.io/rewrite-target: /$2
  ingressClassName: "nginx"
  tls: true

# not needed in this setup
serviceAccount:
  create: false

# not needed in this setup
networkPolicy:
  enabled: false

# CUSTOM: Here you store non secret env vars that will be passed to the pod's environment
extraConfigMapEnvVars: {}

# this is needed to tell the container to read extra env vars from the dedicated Config Map that we create in extraDeploy
extraEnvVarsCM: "{{ include \"common.names.fullname\" . }}"

podAnnotations:
  # this is needed to trigger re-deploy when only Config Map is changed
  # that is the file name generated by extraDeploy nginx/templates/extra-list.yaml
  # that is why it looks so weird in the file path to calculate hash by its content
  checksum/config: "{{ include (print $.Template.BasePath \"/extra-list.yaml\") . | sha256sum }}"

extraDeploy:
  # this creates Config Map from extraConfigMapEnvVars to be able to calculate checksum/config hash by env vars
  - |
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: {{ include "common.names.fullname" . }}
      namespace: {{ include "common.names.namespace" . | quote }}
      labels: {{- include "common.labels.standard" . | nindent 6 }}
        {{- if .Values.commonLabels }}
        {{- include "common.tplvalues.render" ( dict "value" .Values.commonLabels "context" $ ) | nindent 6 }}
        {{- end }}
      {{- if .Values.commonAnnotations }}
      annotations: {{- include "common.tplvalues.render" ( dict "value" .Values.commonAnnotations "context" $ ) | nindent 6 }}
      {{- end }}
    data:
      {{- if .Values.extraConfigMapEnvVars }}
      {{- include "common.tplvalues.render" ( dict "value" .Values.extraConfigMapEnvVars "context" $ ) | trim | nindent 6 }}
      {{- end }}
```

- сделайте commit и push с изменениями в обоих репозиториях.

После этого в github в репозитории вашего сервиса во вкладке меню “Actions” появится новый `workflow run` с вашим коммитом, где должно быть отображено успешное его прохождение

После этого запускаем local-env по [инструкции с readme](https://github.com/TourmalineCore/inner-circle-local-env/blob/master/README.md). При включении local-env в списке поднятых сервисов будет наименование нового добавленного сервиса. Переходим по url local-env и тестим там наш сервис :)

## Чтобы задеплоить новый сервис нам нужно:

- cоздайте папку `.github/workflows` и в нем создайте/обновите файл `prod-docker-publish.yml` (если у вас уже в этой папке есть файл `docker-build-and-push.yml`, просто переименуйте его и дополните)

```
name: Publish Docker image and deploy

on:
 push:
   branches:
     - master
 pull_request:

jobs:
 push_to_registry:
   name: Push Docker image to GitHub
   runs-on: ubuntu-22.04
   permissions:
     packages: write
     contents: read
     attestations: write
   steps:
     - name: Check out the repo
       uses: actions/checkout@v4

     - name: Log in to GitHub Container Registry
       uses: docker/login-action@v2
       with:
         registry: ghcr.io
         username: ${{ github.actor }}
         password: ${{ secrets.GITHUB_TOKEN }}

     - name: Build and push Docker image
       uses: docker/build-push-action@v5
       with:
         context: .
         file: ./Dockerfile
         push: true
         tags: ghcr.io/<your-org-name>/<your-app-name>/<your-service-name>:${{ github.sha }}
         # for example: tags: ghcr.io/tourmalinecore/inner-circle/layout-ui:${{ github.sha }}
 deploy-to-prod-k8s:
   needs: push_to_registry
   name: Deploy service to k8s for prod environment 
   if: github.event_name == 'push'
   runs-on: ubuntu-22.04
   steps:
     - name: checkout
       uses: actions/checkout@v1
     - name: Deploy
       uses: WyriHaximus/github-action-helm3@v3
       with:
         # for example: RELEASE_NAME=layout
         exec: |
           RELEASE_NAME=<your-service-name> 
           helm repo add bitnami https://charts.bitnami.com/bitnami
           helm upgrade --install --namespace dev-inner-circle --create-namespace --values ./ci/values.yaml \
           --set "image.tag=${{ github.sha }}" \
           --set "ingress.enabled=true" \
           --set "ingress.hostname=${{ secrets.DEV_HOST }}" \
           --set "extraConfigMapEnvVars.API_ROOT_AUTH=${{ secrets.DEV_LINK_TO_API_ROOT_AUTH }}" \
           "${RELEASE_NAME}" \
           bitnami/nginx --version 15.0.2
         kubeconfig: "${{ secrets.DEV_KUBECONFIG }}"
```

при деплое в тегах `Build and push Docker image` не воспринимается `latest`, поэтому используем `${{ github.sha }}`

- в папке `ci` добавляем файл `values.yaml`, он точно такой же как ранее добавленный файл `values-local-env.yaml`, отличие только в заданных ресурсах

```
resources:
  limits:
    cpu: 150m
    memory: 300Mi
  requests:
    cpu: 1m
    memory: 25Mi
```

- сделайте commit и push изменений, создайте с ними pull request. После этого в github в репозитории вашего сервиса во вкладке меню “Actions” появится новый `workflow run` с вашим коммитом, где должно быть отображено успешное его прохождение

- далее заходите на прод вашего сервиса и тестируете его работу. Логи подключенного сервиса можно увидеть в Lens в логах пода сервиса, если у вас есть доступ к кластеру
