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
///----- ABBREVIATED CODE -----///

  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "lint": "./node_modules/.bin/nx lint && ng lint",
    "e2e": "ng e2e",
    "affected:apps": "./node_modules/.bin/nx affected:apps",
    "affected:build": "./node_modules/.bin/nx affected:build",
    "affected:e2e": "./node_modules/.bin/nx affected:e2e",
    "affected:dep-graph": "./node_modules/.bin/nx affected:dep-graph",
    "format": "./node_modules/.bin/nx format:write",
    "format:write": "./node_modules/.bin/nx format:write",
    "format:check": "./node_modules/.bin/nx format:check",
    "update": "./node_modules/.bin/nx update",
    "update:check": "./node_modules/.bin/nx update:check",
    "update:skip": "./node_modules/.bin/nx update:skip",
    "workspace-schematic": "./node_modules/.bin/nx workspace-schematic",
    "dep-graph": "./node_modules/.bin/nx dep-graph",
    "postinstall": "./node_modules/.bin/nx postinstall",
    "help": "./node_modules/.bin/nx help",
    "server": "ts-node ./server/server.ts",
    "customer-portal": "ng serve -a=customer-portal -p=4200",
   "admin-portal": "ng serve -a=admin-portal -p=4201"
  },
  
  
///----- ABBREVIATED CODE -----///
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add auth module to the new app

{% hint style="danger" %}
Do not forget to remove the store freeze 
{% endhint %}

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

* Add the LayoutComponent logic to select the current logged in user like in the customer-portal

```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState, getUser } from '@demo-app/auth';
import { User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';

@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.scss']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;

  constructor(private store: Store<AuthState>) {}

  ngOnInit() {
    this.user$ = this.store.select(getUser);
  }
}

```

* Add the material Module to the LayoutModule and export the Layout Component out of the module.

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/layout/src/layout.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { LayoutComponent } from './containers/layout/layout.component';
import { MaterialModule } from '@demo-app/material';

const COMPONENTS = [LayoutComponent];

@NgModule({
  imports: [CommonModule, MaterialModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS]
})
export class LayoutModule {}

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

* Add the Layout Module to the AppModule.

{% hint style="danger" %}
Do not forget the browser animations module for Materials dependency and the basic routes
{% endhint %}

{% hint style="info" %}
NOTE: When we re-use a module/lib we need to manually configure the routing
{% endhint %}

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
import { AuthModule, authRoutes, AuthGuard } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { LayoutModule } from '@demo-app/admin-portal/layout';

@NgModule({
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    StoreModule.forRoot({}),
    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule,
    AuthModule,
    LayoutModule,
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
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}

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

