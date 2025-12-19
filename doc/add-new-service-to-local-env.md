## Чтобы добавить новый сервис в local-env нам нужно:

### В репозиториии [local-env](https://github.com/TourmalineCore/inner-circle-local-env):

- добавить информацию о новом сервисе в файле `deploy/helmfile.yaml`

Пример:

```
  - name: inner-circle-items-api
    labels:
      app: inner-circle-items-api
    wait: true
    chart: bitnami/aspnet-core
    version: 4.4.7
    # it won't work anyway until ingress controller is created
    # thus we wait for it to be ready first
    needs: 
      - ingress-nginx
      - inner-circle-employees-api
      - postgresql
    values:
      # https://helmfile.readthedocs.io/en/latest/#loading-remote-environment-values-files
      - git::https://github.com/TourmalineCore/inner-circle-items-api.git@/ci/values.yaml?ref=master
      - values.yaml.gotmpl
      - values-items-api.yaml.gotmpl
```

- и добавить файл [values-items-api.yaml.gotmpl](https://github.com/TourmalineCore/inner-circle-local-env/blob/master/deploy/values-items-api.yaml.gotmpl) вашего сервиса с нужными переменными

### В сервисе, который подключаем в local-env, в нашем примере [items-api](https://github.com/TourmalineCore/inner-circle-items-api)

- создайте или обновите [Dockerfile](https://github.com/TourmalineCore/inner-circle-items-api/blob/master/Api/Dockerfile), он понадобится для сборки образа и дальнейшего его использования в local-env

- создайте папку `.github/workflows`, если у вас ее еще нет, и добавить в нее файл [.reusable-docker-build-and-push.yml](https://github.com/TourmalineCore/inner-circle-items-api/blob/master/.github/workflows/.reusable-docker-build-and-push.yml) для сборки образа и его публикации в GitHub Container Registry

- также в корне проекта нужно создать папку `ci`, если у вас ее еще нет, и добавить в нее [helmfile.yaml](https://github.com/TourmalineCore/inner-circle-items-api/blob/master/ci/helmfile.yaml)

- в папке `ci` также добавляем файл [values.yaml](https://github.com/TourmalineCore/inner-circle-items-api/blob/master/ci/values.yaml), где описываем настройки параметров Helm-чарта добавляемого сервиса

- сделайте commit и push с изменениями в обоих репозиториях и откройте пулл реквесты

После этого в ваших пулл реквестах запустятся новые `workflow`, где должно быть отображено успешное их прохождение

После этого запускаем local-env по [инструкции с readme](https://github.com/TourmalineCore/inner-circle-local-env/blob/master/README.md). В кластере local-env в списке поднятых сервисов будет наименование нового добавленного сервиса. Переходим по url local-env и тестим там наш сервис :)

## Чтобы задеплоить новый сервис нам нужно:

- в `.github/workflows` создайте/обновите файл [deploy-to-prod-from-default.yml](https://github.com/TourmalineCore/inner-circle-items-api/blob/master/.github/workflows/deploy-to-prod-from-default.yml)

- сделайте commit и push изменений, создайте с ними pull request. После этого в ваших пулл реквестах запустятся новые `workflow`, где должно быть отображено успешное их прохождение

- далее заходите на прод вашего сервиса и тестируете его работу
