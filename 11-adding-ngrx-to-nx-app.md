---
description: >-
  In this section we will add and discuss the default addition of NgRx to root
  and feature modules.
---

# 11 - Adding NgRx to Nx App

## 1. Add NgRx to Customer Portal App

* Add a default set up for ngrx to our new app. We can run the generate command for ngrx with the module and --onlyEmptyRoot option to only add the StoreModule.forRoot and EffectsModule.forRoot calls without generating any new files versus --root which will make a default reducer and effect.

```text
ng g ngrx app --module=apps/customer-portal/src/app/app.module.ts  --onlyEmptyRoot
```

{% hint style="info" %}
Due to issue with StoreFreeze make sure you comment out this line of code
{% endhint %}

{% code title="apps/customer-portal/src/app/app.module.ts" %}
```typescript
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    NxModule.forRoot(),
    RouterModule.forRoot(
      [
        { path: '', pathMatch: 'full', redirectTo: 'products' },
        { path: 'auth', children: authRoutes },
        {
          path: 'products',
          loadChildren: '@demo-app/products#ProductsModule',
          canActivate: [AuthGuard]
        }
      ],
      { initialNavigation: 'enabled' }
    ),
    AuthModule,
    LayoutModule,
    StoreModule.forRoot({}
      // ,{ metaReducers : !environment.production ? [storeFreeze] : [] }
    ),
    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule
  ],
```
{% endcode %}

## 2. Add NgRx Auth lib making it a state state

```text
ng generate ngrx auth --module=libs/auth/src/lib/auth.module.ts
```

![New Nx Lib with State folder](.gitbook/assets/image%20%2823%29.png)

