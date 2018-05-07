# 11 - Adding nx workspaces

## 1. Add another app to our nx workspace called admin-portal

```text
ng g app admin-portal --style scss --routing
```

* add default ngrx setup to our new app

```text
ng g ngrx app --module=apps/admin-portal/src/app/app.module.ts  --onlyEmptyRoot
```

## 2. Add scripts to start each app more easily to package.json

_**package.json**_

```javascript
"scripts": {
   ...
   "customer-portal": "ng serve -a=customer-portal -p=4200",
   "admin-portal": "ng serve -a=admin-portal -p=4201",
   ...
}
```

* Add auth module to the new app

```typescript
@NgModule({
  imports: [
  ...
  AuthModule.forRoot()
  ...
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
  providers: [AppEffects]
})
export class AppModule { }
```

## 3. Add a new layout lib and component

```text
ng g lib layout --directory=admin-portal
```

* Add a layout container component

```text
ng g c containers/layout -a=admin-portal/layout
```

* Add a material tool bar

_**libs/admin-portal/layout/src/containers/layout/layout.component.html**_

```markup
<mat-toolbar color="primary" fxLayout="row">
<span>Admin Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```

* Add the new component to the admin-portal apps main view

_**apps/admin-portal/src/app/app.component.html**_

```markup
<app-layout>
    <router-outlet></router-outlet>
</app-layout>
```

* Add styles to styles.scss

_**apps/admin-portal/src/styles.scss**_

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```

* Do not forget the browser animations module for Materials dependency and the basic routes

NOTE: When we re-use a module we need to manually configure the routing

_**apps/admin-portal/src/app/app.module.ts**_

```typescript
@NgModule({
  imports: [
  BrowserModule,
  NxModule.forRoot(),
  RouterModule.forRoot(
  [
    { path: '', pathMatch: 'full', redirectTo: 'user-profile' },
    { path: 'auth', children: authRoutes },
    {
      path: 'user-profile',
      loadChildren: '@demo-app/user-profile#UserProfileModule',
      canActivate: [AuthGuard]
    }
  ]), 
  BrowserAnimationsModule,
  ...
```

* As we did not generate the Auth module to sync with the new app we need to manually register the lazy loaded parts

_**apps/admin-portal/src/tsconfig.app.json**_

```typescript
{
  "extends": "../../../tsconfig.json",
  "compilerOptions": {
    "outDir": "../../../dist/out-tsc/apps/admin-portal",
    "module": "es2015"
  },
  "include": [
    "**/*.ts"
    /* add all lazy-loaded libraries here: "../../../libs/my-lib/index.ts" */
    , "../../../libs/user-profile/index.ts"
  ],
  "exclude": [
    "**/*.spec.ts"
  ]
}
```

