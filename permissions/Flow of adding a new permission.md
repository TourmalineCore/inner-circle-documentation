# Flow of adding a new permission

## 1 step - accounts-api

В репозитории [accounts-api](https://github.com/TourmalineCore/accounts-api):

1. В файл `Core/Models/Permissions.cs`

```c#
public const string CanManageExample = "CanManageExample";
```

2. В файл `DataAccess/Mapping/MappingData.cs`

```c#
new Permission(Permissions.CanManageExample),
```

3. Накатить миграцию на добавленную пермисиию, как это сделать, можно посмотреть в `README.md`

## 2 step - accounts-ui

В репозитории [accounts-ui](https://github.com/TourmalineCore/accounts-ui):

1. В файл `src/features/account-management/roles-enums.ts`

```ts
CanManageExample: 'Can Manage Example',
```

2. В файл `src/routes/state/AccessBasedOnPemissionsState.ts`

```ts
CanManageExample = 'CanManageExample',
```

3. В файл `src/features/account-management/RolesPageContent.tsx` можно добавить либо новую группу пермисиий, либо же уже в существующую группу добавить нужную пермиссию

```ts
{
    groupName: 'Example',
    children: [
        { id: 'CanManageExample', name: 'Can Manage Example' },
    ],
},
```

>[пример](https://github.com/TourmalineCore/accounts-ui/commit/482ebc3288c3ab22786d10e4f79cb1932bc390e1) коммита с добавлением новой пермиссии в существующую группу аккаунтов 

>[пример](https://github.com/TourmalineCore/accounts-ui/commit/9d94cafc453fc24f30f57b26d3b4d12b22ae33ec) коммита с добавлением новой группы книг со всеми новыми пермиссиями

## 3 step - ваш API

1. В файл `Api/UserClaimsProvider.cs`

```c#
public const string CanManageExample = "CanManageExample";
```

## 4 step - ваш UI - два варианта развития событий

### Если добавляете пермисии в сервисы с подключенным layout-ui (compensations-ui или books-ui)

#### В [layout-ui](https://github.com/TourmalineCore/inner-circle-layout-ui)

1. В файл `src/routes/state/AccessBasedOnPemissionsState.ts` в enum Permission добавить

```ts
CanManageExample = `CanManageExample`,
```

#### В вашем UI

1. Для отображения страницы в зависимости от пермиссий, в файле `src/routes/pageRoutes.tsx` обернуть нужный роут в проверку нужной пермиссии

```tsx
if (accessPermissions.get(`CanManageExample`)) {
  routes.push(...exampleRoutes)
}
```

2. (Не обязательно) Если нужно, чтобы в вашем UI сервсие отдельный компонент внутри страницы отображался в зависимости от пермиссий, то нужно 

- Установить в сервис "jwt-decode": "^4.0.0"

- Добавить файл `src/common/hasAccessPermission.ts`

```ts
import { useContext } from "react"
import { jwtDecode } from 'jwt-decode'
import { authService } from "./authService"

interface DecodedToken {
  permissions: string[],
}

export const hasAccessPermission = ({
  permission,
}: {
  permission: string,
}): boolean => {
  // eslint-disable-next-line react-hooks/rules-of-hooks
  const authContext = useContext<string[]>(authService.AuthContext)

  const token = authContext?.[0] || null

  if (!token) {
    return false
  }

  try {
    const decodedToken: DecodedToken = jwtDecode<DecodedToken>(token)

    return decodedToken.permissions.includes(permission)
  }
  catch (error) {
    // eslint-disable-next-line no-console
    console.error(`Error decoding token:`, error)
    return false
  }
}
```

- И обернуть его, как компонент `<Actions />`

```tsx
return (
  <>
      {
         hasAccessPermission({
           permission: `CanManageExample`,
         }) && <Actions />
      }
  </>
  )
```

### Если добавляете пермисии в сервисы где еще нет layout-ui (inner-circle-ui или inner-circle-documents)

1. В вашем UI сервсие в файл `src/routes/state/AccessBasedOnPemissionsState.ts` в enum Permission добавить

```ts
CanManageExample = `CanManageExample`,
```

2. В файлe `src/routes/adminRoutes.tsx` в функции `getAdminRoutes` обернуть нужный роут в проверку нужной пермиссии

```tsx
if (accessPermissions.get('CanManageExample')) {
  routes.push(...exampleRoutes);
}
```
