## Предисловие
Мы [работаем над обновлением](https://github.com/TourmalineCore/React-Packages/pull/52) монорепозитория React-Packages, а именно [пакета аутентификации в нём](https://github.com/TourmalineCore/React-Packages/tree/master/packages/react-tc-auth). Чтобы удобно и легко протестировать наш пакет с обновлениями локально, мы хотели подключить его к [сервису компенсаций](https://github.com/TourmalineCore/inner-circle-compensations-ui) с помощью `npm link`

## Проблема в работе с `npm link`

Сама логика использования `npm link` проста и понятна:  
1. В терминале пакета, который хотим подключить (в нашем случае это [пакет аутентификации react-tc-auth](https://github.com/TourmalineCore/React-Packages/tree/master/packages/react-tc-auth)), вызываем команду `npm link` в папке, где находится пакетный package.json

2. В терминале сервиса (в нашем случае это [сервис компенсаций](https://github.com/TourmalineCore/inner-circle-compensations-ui)), куда хотим подключить пакет, вводим `npm link @myorg/privatepackage`, имя пакета берем из поля name в его  package.json

Таким образом пакет получилось подключить, но с одной проблемой - странной ошибкой о хуках,
![Error text](./images/error-text.png)
которая не говорила о конкретной проблеме и очень настойчиво и неизменно игнорировала наши изменения в коде, которые мы вносили, в попытках это пофиксить...

В данном [обсуждении](https://stackoverflow.com/questions/56663785/invalid-hook-call-hooks-can-only-be-called-inside-of-the-body-of-a-function-com)  мы впервые прочитали о связи этой ошибки с npm link и дублированием реакта, но предложенные решения нам не помогли.

Позже получилось найти [одно решение](https://github.com/facebook/react/issues/13991#issuecomment-1867954003), где предлагалось в node\_modules подключенного с помощью `npm link` пакета удалить React, и с помощью `npm link` подключить React из node\_modules сервиса, в который подключили пакет (5 шаг в Solution), чтобы исключить дублирование реактов.

Мы решили протестировать данное решение, в надежде, что оно нам поможет и смогли прийти к рабочему результату.

Чтобы все заработало как надо в нашем случае, нужно провести несколько этапов танцев с бубнами:  
terminal in [react-tc-auth](https://github.com/TourmalineCore/React-Packages/tree/master/packages/react-tc-auth) folder

1. удалить папки node\_modules и es  
2. nvm use 12 (без этого шаг 3 будет падать с проблемой выполнения npx simple-git-hooks)  
3. npm ci

terminal in [React-Packages](https://github.com/TourmalineCore/React-Packages) folder

5. npm ci (для установки общих пакетов для выполнения билда пакетов)

terminal in [react-tc-auth](https://github.com/TourmalineCore/React-Packages/tree/master/packages/react-tc-auth) folder

5. npm run build   
6. nvm use 20.18 (повышаем ноду, чтобы link хранился в папке той же ноды, какая используется в проекте, куда подключаем пакет)  
7. npm link   

В [сервисе](https://github.com/TourmalineCore/inner-circle-compensations-ui), куда подключаем пакет:

1. npm link @tourmalinecore/react-tc-auth  
2. находим в node\_modules подключенный пакет и открываем node\_modules пакета, открываем терминал этой папки  
3. rm \-rf react  
4. rm \-rf react-dom  
5. rm \-rf react-router-dom                                                
6. ln \-s *your\_full\_path*/inner-circle-compensations-ui/node\_modules/react  
7. ln \-s *your\_full\_path*/inner-circle-compensations-ui/node\_modules/react-dom  
8. ln \-s *your\_full\_path*/inner-circle-compensations-ui/node\_modules/react-router-dom  
   

После этого ошибка о хуках больше не отображается и мы видим сервис с подключенным пакетом через npm link в рабочем состоянии![Working compensation service](./images/working-compensation-service.png)
