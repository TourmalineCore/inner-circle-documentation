# Layering

In our project we follow the pattern of **Container/Content** controlled by MobX state.
To follow this approach we single out independent sections on the page which have their own state and network calls.

Below is a typical folder structure for this approach. In this case _auth_ is the only section on the page, so we don't need to create a _sections_ folder:

```bash
├── pages
│   ├── auth
│   │   └── state
│   │       └── AuthState.cy.ts
│   │       └── AuthState.ts
│   │       └── AuthStateContext.ts
│   │   └── AuthContainer.tsx
│   │   └── AuthContent.tsx
│   │   └── AuthPage.scss
│   │   └── AuthPage.tsx
```

*Page.tsx file only wraps the Container in context provider, e.g.:

```javascript
AuthPage.tsx

export function AuthPage() {
  const authState = useMemo(
    () => new AuthState(),
    [],
  )

  return (
    <AuthStateContext.Provider value={authState}>
      <AuthContainer />
    </AuthStateContext.Provider>
  )
}
```

## Pros of Container/Content Pattern with MobX

1. Separation of concerns
- Container handles business logic and API calls
- Content handles presentation, rendering on UI 
2. Testability
- Container can be tested with integration tests (API mocking)
- Content can be tested with unit tests (props, rendering)
- State can be tested independently with Cypress
3. State management
- Centralized state logic
- Automatic re-rendering when state changes
4. Maintainability 
- Easier to debug and refactor

## Cons of Container/Content Pattern with MobX

1. Boilerplate code
2. Complexity
- More files to navigate
- Higher learning curve, because you need more time for onboarding
- Simple components become over-engineered

## When to Use
This approach is worth following for complex components with state and API calls. Also if testing is of a high priority on the project, the layering approach is quite useful to separate different testing types. 
In case of simple static components this approach might be an overkill. 