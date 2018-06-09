# 14 - NgRx Selectors

Lets add a main menu to our customer portal to show the name of the logged in user.

1. **Add a new layout lib to a customer-portal directory**

```text
ng g lib layout --directory=customer-portal
```

## 2. Add a layout container component

```text
ng g c containers/layout -a=customer-portal/layout
```

## 3. Add Material and Router module

{% code-tabs %}
{% code-tabs-item title="libs/customer-portal/layout/src/layout.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MaterialModule } from '@demo-app/material';
import { LayoutComponent } from './containers/layout/layout.component';
import { RouterModule } from '@angular/router';

const COMPONENTS = [LayoutComponent];
@NgModule({
  imports: [CommonModule, MaterialModule, RouterModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS]
})
export class LayoutModule {
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Add a material toolbar

{% code-tabs %}
{% code-tabs-item title="libs/customer-portal/layout/src/containers/layout/layout.component.html" %}
```typescript
<mat-toolbar color="primary" fxLayout="row">
  <span>Customer Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add styles to styles.scss

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/styles.scss" %}
```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="libs/customer-portal/layout/src/containers/layout/layout.component.scss" %}
```css
.right-nav {
    margin-left: auto;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 6. Update layout component to select user from the store

{% code-tabs %}
{% code-tabs-item title="libs/customer-portal/layout/src/containers/layout/layout.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState } from '@demo-app/auth';
import { User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';

@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.scss']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;

  constructor(private store: Store<AuthState>) { }

  ngOnInit() {
    this.user$ = this.store.select(state => state.auth.user);
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add layout module to the customer-portal app

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/app/app.module.ts" %}
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
import { authRoutes, AuthModule } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { AuthGuard } from '@demo-app/auth';
import { LayoutModule } from '@demo-app/customer-portal/layout';


@NgModule({
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    BrowserAnimationsModule,
    RouterModule.forRoot([
      { path: 'auth', children: authRoutes },
      { path: 'user-profile', loadChildren: '@demo-app/user-profile#UserProfileModule', canActivate: [AuthGuard] }
    ], {
        initialNavigation: 'enabled'
      }),
    //StoreModule.forRoot({},{ metaReducers : !environment.production ? [storeFreeze] : [] }),
    StoreModule.forRoot({}),
    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule,
    AuthModule,
    LayoutModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the layout component to the main app.component.html

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/app/app.component.html" %}
```markup
<app-layout>
    <router-outlet></router-outlet>
</app-layout>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 7. Add selector file

* Add a file called index.ts to the +state folder of your auth state lib

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/index.ts" %}
```typescript
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { AuthData } from './auth.reducer';

export const getAuthState = createFeatureSelector<AuthData>('auth');
export const getUser = createSelector(getAuthState, state => state.user);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Ensure you have re-exported your publically available paths in the auth libs index.ts file

{% code-tabs %}
{% code-tabs-item title="libs/auth/index.ts" %}
```typescript
export { AuthGuard } from './src/guards/auth/auth.guard';
export { AuthModule, authRoutes } from './src/auth.module';
export { AuthState } from './src/+state/auth.reducer';
export * from './src/+state';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 8. Use selector in Layout component

{% code-tabs %}
{% code-tabs-item title="libs/customer-portal/layout/src/containers/layout/layout.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState, getUser } from '@demo-app/auth';
import { User } from '@demo-app/data-models';
import { Observable
 } from 'rxjs/Observable';

@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.scss']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;
  constructor(private store: Store<AuthState>) { }

  ngOnInit() {
    this.user$ = this.store.select(getUser);
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

