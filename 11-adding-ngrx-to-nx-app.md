---
description: >-
  In this section we will add and discuss the default addition of NgRx to root
  and feature modules.
---

# 11 - Adding NgRx to Nx App

## 1. Add NgRx to Customer Portal App

- Add a default set up for ngrx to our new app. We can run the generate command for ngrx with the module and --minimal true option to only add the StoreModule.forRoot and EffectsModule.forRoot calls without generating any new files versus --root which will make a default reducer and effect.

```text
nx g @nrwl/angular:ngrx --module=apps/customer-portal/src/app/app.module.ts  --minimal true
? What name would you like to use for the NgRx feature state? An example would be "users". products
? Is this the root state of the application? Yes
? Would you like to use a Facade with your NgRx state? No
✔ Packages installed successfully.
UPDATE apps/customer-portal/src/app/app.module.ts
UPDATE package.json
```

Here are the changes to the app.module.ts:

{% code title="apps/customer-portal/src/app/app.module.ts" %}

```typescript
imports: [
    BrowserModule,
    BrowserAnimationsModule,
    NxModule.forRoot(),
    RouterModule.forRoot(
      /// abbreviated 
    ),
    AuthModule,
    LayoutModule,
    StoreModule.forRoot(
      {},
      {
        metaReducers: !environment.production ? [] : [],
        runtimeChecks: {
          strictActionImmutability: true,
          strictStateImmutability: true,
        },
      }
    ),
    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule.forRoot(),
  ],
```

{% endcode %}

## 2. Add NgRx Auth lib making it a state state

```text
nx g @nrwl/angular:ngrx --module=libs/auth/src/lib/auth.module.ts --minimal false
? What name would you like to use for the NgRx feature state? An example would be "users". auth
? Is this the root state of the application? No
? Would you like to use a Facade with your NgRx state? No
CREATE libs/auth/src/lib/+state/auth.actions.ts
CREATE libs/auth/src/lib/+state/auth.effects.spec.ts
CREATE libs/auth/src/lib/+state/auth.effects.ts
CREATE libs/auth/src/lib/+state/auth.models.ts
CREATE libs/auth/src/lib/+state/auth.reducer.spec.ts
CREATE libs/auth/src/lib/+state/auth.reducer.ts
CREATE libs/auth/src/lib/+state/auth.selectors.spec.ts
CREATE libs/auth/src/lib/+state/auth.selectors.ts
UPDATE libs/auth/src/lib/auth.module.ts
UPDATE libs/auth/src/index.ts
```

![New Nx Lib with State folder](.gitbook/assets/image%20%2823%29.png)
