## Чтобы добавить новый сервис в local-env нам нужно:

### В репозиториии local-env:

- добавить в `helmfile.yaml` в папке `deploy`

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
     - <your-service-name>.yaml.gotmpl
```

- создать файл `<your-service-name>.yaml.gotmpl` в папке `deploy`  
пример содержимого файла:

```
extraConfigMapEnvVars:
  ENV_KEY: "\"local\""
  API_ROOT: "\"{{ .Values.baseExternalUrl }}/api\""
  API_ROOT_AUTH: "\"{{ .Values.baseExternalUrl }}/auth-api\""
```

### В своем сервисе

- настройте `local-config-builder.js`, если его у вас нет

```
import fs from "fs"
const env = process.argv[2]


const filepath = "./public/env-config.js"
const fileCypressPath = "./cypress/env-config.js"
const data = fs.readFileSync(`./.config-${env}`)


const variables = data.toString()
 .split("\n")
 .map((str) => {
   const regex = /^([^:]+):(.+)/gm
   const m = regex.exec(str)
   return `${m[1].trim()}: "${m[2].trim()}",`
 })
 .reduce((res, x) => res.concat(x), "")


fs.writeFileSync(filepath, `window.__ENV__ = { ${variables} }`)
fs.writeFileSync(fileCypressPath, `window.__ENV__ = { ${variables} }`)
```

- и добавьте `env-config.js` в `.gitignore`:

```
**/*/env-config.js
```

- не забудьте добавить в `index.html` строку 

```
<script src="/env-config.js"></script>
```

- обновите `package.json`:

```
"start": "run-s create-config:prod vite-start",
"start:local": "run-s create-config:local vite-start",
"vite-start": "vite --host",
"create-config:prod": "node local-config-builder prod",
"create-config:local": "node local-config-builder local",
```

- создайте или обновите `Dockerfile`

```
FROM node:20.11.1-alpine3.19 as build
COPY package.json .
COPY package-lock.json .
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:1.26.0-alpine3.19-slim
COPY /ci/nginx.conf /data/conf/nginx.conf
COPY --from=build /dist /usr/share/nginx/html

EXPOSE 80

WORKDIR /usr/share/nginx/html
COPY ./ci/env.sh .
COPY .env-vars .
RUN apk add --no-cache bash
RUN chmod +x /usr/share/nginx/html/env.sh

CMD /bin/bash -c "/usr/share/nginx/html/env.sh" && nginx -g "daemon off;" -c "/data/conf/nginx.conf"
```

- создайте папку `.github/workflows` и в нем файл `docker-build-and-push.yml`

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
         tags: ghcr.io/<your-org-name>/<your-app-name>/<your-service-name>:${{ github.sha }}
```

- сделайте commit и push с изменениями в обоих репозиториях.

После этого в github в репозитории вашего сервиса во вкладке меню “Actions” появится новый `workflow run` с вашим коммитом, где должно быть написано “Push Docker image to Docker Hub succeeded”

После этого запускаем local-env по [инструкции с readme](https://github.com/TourmalineCore/inner-circle-local-env/blob/master/README.md). При включении в списке поднятых сервисов в local-env увидим наименование нового добавленного сервиса. Переходим по url local-env и тестим там наш сервис :)

## Чтобы задеплоить новый сервис нам нужно:

- cоздайте папку `.github/workflows` и в нем создайте/обновите файл `prod-docker-publish.yml` (если у вас уже в этой папке есть файл `docker-build-and-push.yml`, просто переименуйте его и дополните)

```
name: Publish Docker image and deploy <your-service-name>

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
         exec: |
           RELEASE_NAME=<your-service-name>
           helm repo add bitnami https://charts.bitnami.com/bitnami
           helm upgrade --install --namespace dev-inner-circle --create-namespace --values ./ci/values.yaml \
           --set "image.tag=${{ github.sha }}" \
           --set "ingress.enabled=true" \
           --set "ingress.hostname=${{ secrets.DEV_HOST }}" \
           --set "extraConfigMapEnvVars.API_ROOT_AUTH=${{ secrets.DEV_LINK_TO_API_ROOT_AUTH }}" \
           --set "extraConfigMapEnvVars.ENV_KEY=${{ secrets.DEV_ENV }}" \
           "${RELEASE_NAME}" \
           bitnami/nginx --version 15.0.2
         kubeconfig: "${{ secrets.DEV_KUBECONFIG }}"
```

при деплое в тегах `Build and push Docker image` не воспринимается `latest`, поэтому используем `${{ github.sha }}`

- добавьте `.config-dev` с нужными для сервиса env переменными

```
ENV_KEY:dev
API_ROOT_AUTH:https://<your-app-name>/api/auth
```

- сделайте commit и push изменений, создайте с ними pull request. После этого в github в репозитории вашего сервиса во вкладке меню “Actions” появится новый `workflow run` с вашим коммитом, где должно быть отображено успешное его прохождение

- далее заходите на прод вашего сервиса и тестируете его работу. Логи подключенного сервиса можно увидеть в Lens в логах пода сервиса, если у вас есть доступ к кластеру

