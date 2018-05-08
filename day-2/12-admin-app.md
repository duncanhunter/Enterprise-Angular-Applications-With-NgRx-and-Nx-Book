# 12 - Admin App

## 1. Add another app to our Nx workspace called admin-portal

```text
ng g app admin-portal --style scss --routing --prefix app
```

* Add default ngrx setup to our new app

```text
ng g ngrx app --module=apps/admin-portal/src/app/app.module.ts  --onlyEmptyRoot
```

## 2. Add scripts to start each app more easily to package.json

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
"scripts": {
   ...
   "customer-portal": "ng serve -a=customer-portal -p=4200",
   "admin-portal": "ng serve -a=admin-portal -p=4201",
   ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add auth module to the new app and remove store freeze

{% code-tabs %}
{% code-tabs-item title="apps/admin-portal/src/app/app.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { BrowserModule } from '@angular/platform-browser';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';
import { StoreRouterConnectingModule } from '@ngrx/router-store';
import { storeFreeze } from 'ngrx-store-freeze';
import { AuthModule } from '@demo-app/auth';

@NgModule({
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    RouterModule.forRoot([], { initialNavigation: 'enabled' }),
    StoreModule.forRoot({}),
    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule,
    AuthModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Add a new layout lib and component

```text
ng g lib layout --directory=admin-portal
```

* Add a layout container component

```text
ng g c containers/layout -a=admin-portal/layout
```

* Add a material tool bar

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/layout/src/containers/layout/layout.component.html" %}
```markup
<mat-toolbar color="primary" fxLayout="row">
<span>Admin Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the new component to the admin-portal apps main view

{% code-tabs %}
{% code-tabs-item title="apps/admin-portal/src/app/app.component.html" %}
```markup
<app-layout>
    <router-outlet></router-outlet>
</app-layout>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add styles to styles.scss

{% code-tabs %}
{% code-tabs-item title="apps/admin-portal/src/styles.scss" %}
```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Do not forget the browser animations module for Materials dependency and the basic routes

NOTE: When we re-use a module we need to manually configure the routing

{% code-tabs %}
{% code-tabs-item title="apps/admin-portal/src/app/app.module.ts" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

* As we did not generate the Auth module to sync with the new app we need to manually register the lazy loaded parts

{% code-tabs %}
{% code-tabs-item title="apps/admin-portal/src/tsconfig.app.json" %}
```typescript
{
  "extends": "../../../tsconfig.json",
  "compilerOptions": {
    "outDir": "..
/../../dist/out-tsc/apps/admin-portal",
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
{% endcode-tabs-item %}
{% endcode-tabs %}

