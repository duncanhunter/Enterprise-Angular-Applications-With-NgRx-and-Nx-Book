---
description: >-
  In this section we will add a layout lib and some share User state with a
  behavior Subject to add to a main menu.
---

# 8 - Layout Lib and BehaviorSubjects

## 1. Add a new layout lib and container component

```text
ng g lib layout --prefix app
```

* Add a layout container component

```text
ng g c containers/layout --project=layout
```

## 2. Add a material tool bar

{% code-tabs %}
{% code-tabs-item title="libs/layout/src/lib/containers/layout/layout.component.html" %}
```markup
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



## 3. Add a BehaviourSubject to Auth service

A BehaviorSubject is a special observable you can both subscribe to and pass values.

* Add a BehaviorSubject to the Auth service.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/lib/services/auth/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private userSubject$ = new BehaviorSubject<User>(null);
  user$ = this.userSubject$.asObservable();

  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate): Observable<User> {
    return this.httpClient.post<User>(
      'http://localhost:3000/login',
      authenticate
    ).pipe(tap((user: User) => (this.userSubject$.next(user))));
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Add the LayoutComponent logic to select the current logged in user like

Re-export the  AuthService from the Auth Libs index.ts file.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/index.ts" %}
```typescript
export { AuthService } from './lib/services/auth/auth.service';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add logic to subscribe to User Subject in the Auth Service

{% code-tabs %}
{% code-tabs-item title="libs/layout/src/lib/containers/layout/layout.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { AuthService } from '@demo-app/auth'
import { Observable } from 'rxjs';
import { User } from '@demo-app/data-models';

@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.css']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;

  constructor(private authService: AuthService) {}

  ngOnInit() {
    this.user$ = this.authService.user$;
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the MaterialModule to the LayoutModule and export the Layout Component out of the module.

{% code-tabs %}
{% code-tabs-item title="libs/layout/src/lib/layout.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { LayoutComponent } from './containers/layout/layout.component';
import { MaterialModule } from '@demo-app/material';

@NgModule({
  imports: [CommonModule, MaterialModule],
  declarations: [LayoutComponent],
  exports: [LayoutComponent]
})
export class LayoutModule {}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the new component to the cutomer-portal apps main view

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
{% code-tabs-item title="apps/customer-portal/src/styles.scss" %}
```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add styles to the layout.component.scss 

{% code-tabs %}
{% code-tabs-item title="libs/layout/src/lib/containers/layout/layout.component.scss" %}
```text
.right-nav {
    margin-left: auto;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the Layout Module to the AppModule.

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/app/app.module.ts" %}
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { AuthGuard } from '@demo-app/auth';
import { LayoutModule } from '@demo-app/layout';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    NxModule.forRoot(),
    RouterModule.forRoot(
      [
        { path: 'auth', children: authRoutes },
      ],
      { initialNavigation: 'enabled' }
    ),
    AuthModule,
    LayoutModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 

### ????. Share User state with a BehaviorSubject



## Extras

### 1. Convert Layout component into a pure container component 

* Add a toolbar presentational component.
* Pass user into presentational component via inputs.

