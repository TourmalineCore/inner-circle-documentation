# Flow of Adding a New Permission

## Step 1 - accounts-api

In the [accounts-api](https://github.com/TourmalineCore/accounts-api) repository:

1. In the file `Core/Models/Permissions.cs` add line

```c#
public const string CanManageExample = "CanManageExample";
```

2. In the file `DataAccess/Mapping/MappingData.cs` add line

```c#
new Permission(Permissions.CanManageExample),
```

3. Roll out the migration for the added permission, instructions on how to do this can be found in `README.md`

## Step 2 - accounts-ui

In the [accounts-ui](https://github.com/TourmalineCore/accounts-ui) repository:

1. In the file `src/features/account-management/roles-enums.ts` add line

```ts
CanManageExample: 'Can Manage Example',
```

2. In the file `src/routes/state/AccessBasedOnPemissionsState.ts` add line

```ts
CanManageExample = 'CanManageExample',
```

3. In the file `src/features/account-management/RolesPageContent.tsx` you can either add a new group of permissions or add the required permission to an existing group

```ts
{
    groupName: 'Example',
    children: [
        { id: 'CanManageExample', name: 'Can Manage Example' },
    ],
},
```

>[Example](https://github.com/TourmalineCore/accounts-ui/commit/482ebc3288c3ab22786d10e4f79cb1932bc390e1) of a commit with adding a new permission to **an existing group** of accounts

>[Example](https://github.com/TourmalineCore/accounts-ui/commit/9d94cafc453fc24f30f57b26d3b4d12b22ae33ec) of a commit with adding a **new group** of books with all new permissions

## Step 3 - Your API

1. In the file `Api/UserClaimsProvider.cs` add line

```c#
public const string CanManageExample = "CanManageExample";
```

## Step 4 - Your UI - Two Possible Scenarios

### First Scenario: If you are adding permissions to services with connected layout-ui (compensations-ui or books-ui)

#### In [layout-ui](https://github.com/TourmalineCore/inner-circle-layout-ui)

1. In the file `src/routes/state/AccessBasedOnPemissionsState.ts` add line to the Permission enum

```ts
CanManageExample = `CanManageExample`,
```

#### In Your UI

1. To display a page based on permissions, in the file `src/routes/pageRoutes.tsx` wrap the required route in a check for the necessary permission

```tsx
if (accessPermissions.get(`CanManageExample`)) {
  routes.push(...exampleRoutes)
}
```

2. (Optional) If you need a separate component within the page of your UI service to be displayed based on permissions, you need to:

- Install "jwt-decode": "^4.0.0"

- Add the file `src/common/hasAccessPermission.ts`

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

- And wrap it as a component `<Actions />`

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

### If you are adding permissions to services where layout-ui is not yet available (inner-circle-ui or inner-circle-documents)

1. In your UI service, in the file `src/routes/state/AccessBasedOnPemissionsState.ts` add line to the Permission enum

```ts
CanManageExample = `CanManageExample`,
```

2. In the file `src/routes/adminRoutes.tsx` in the function `getAdminRoutes` wrap the required route in a check for the necessary permission

```tsx
if (accessPermissions.get('CanManageExample')) {
  routes.push(...exampleRoutes);
}
```
