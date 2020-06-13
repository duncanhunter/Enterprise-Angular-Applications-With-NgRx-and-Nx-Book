---
description: >-
  In this section we make a reusable Layout and a BehaviorSubject to share User
  state
---

# 8 - Layout Lib and BehaviorSubjects

## 1. Add a new Layout lib and container component

```text
nx generate @nrwl/angular:lib layout
? Which stylesheet format would you like to use? SASS(.scss)  [ http://sass-lang.com   ]
```

Again, choose sass for the stylesheet language.

* Add a layout container component

```text
ng g @nrwl/angular:component containers/layout --project=layout --prefix app
```

## 2. Add the MaterialModule and Router Module to the LayoutModule and export the Layout Component out of the module

{% code title="libs/layout/src/lib/layout.module.ts" %}

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { LayoutComponent } from './containers/layout/layout.component';
import { MaterialModule } from '@demo-app/material';    // Added
import { RouterModule } from '@angular/router';   // Added

@NgModule({
  imports: [CommonModule, MaterialModule, RouterModule], // Added
  declarations: [LayoutComponent],
  exports: [LayoutComponent]
})
export class LayoutModule {}

```

{% endcode %}

## 3. Add a material tool bar to the layout component

Note the &lt;ng-content&gt;&lt;/ng-content&gt; to transclude content into the component. Also note the flexLayout attribute.

{% code title="libs/layout/src/lib/containers/layout/layout.component.html" %}

```markup
<mat-toolbar color="primary" fxLayout="row">
  <span>Customer Portal</span>
</mat-toolbar>
<ng-content></ng-content>
```

{% endcode %}

## 3. Add a BehaviourSubject to Auth service

A BehaviorSubject is a special observable you can both subscribe to and pass values.

* Add a BehaviorSubject to the Auth service.

{% code title="libs/auth/src/lib/services/auth/auth.service.ts" %}

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
    return this.httpClient
      .post<User>('http://localhost:3000/login', authenticate)
      .pipe(tap((user: User) => this.userSubject$.next(user)));
  }
}

```

{% endcode %}

## 4. Add the LayoutComponent logic to select the current logged in user like

Re-export the  AuthService from the Auth Libs index.ts file.

{% code title="libs/auth/src/index.ts" %}

```typescript
export { AuthService } from './lib/services/auth/auth.service';
```

{% endcode %}

Add logic to subscribe to User Subject in the Auth Service

{% code title="libs/layout/src/lib/containers/layout/layout.component.ts" %}

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

{% endcode %}

## 6. Add the new component to the Customer Portal apps main view

{% code title="apps/customer-portal/src/app/app.component.html" %}

```markup
<app-layout>
    <router-outlet></router-outlet>
</app-layout>
```

{% endcode %}

## 7. Add styles to styles.scss and layout.scss

{% code title="apps/customer-portal/src/styles.scss" %}

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

body {
    margin: 0;
}
```

{% endcode %}

* Add styles to the layout.component.scss 

{% code title="libs/layout/src/lib/containers/layout/layout.component.scss" %}

```text
.right-nav {
    margin-left: auto;
}
```

{% endcode %}

## 8. Add the Layout Module to the AppModule.

{% code title="apps/customer-portal/src/app/app.module.ts" %}

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';   // Added
import { LayoutModule } from '@demo-app/layout';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    NxModule.forRoot(),
    RouterModule.forRoot([{path: 'auth', children: authRoutes}], { initialNavigation: 'enabled' }),
    AuthModule,
    LayoutModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

```

{% endcode %}

## 9. Add a material tool bar logic

We will add the products link later but for now it will error if we try and click it.

{% code title="libs/layout/src/lib/containers/layout/layout.component.html" %}

```markup
<mat-toolbar color="primary" fxLayout="row">
  <span>Customer Portal</span>
  <div class="right-nav">
    <span *ngIf="user$ | async as user">
      <button mat-button>{{(user$ | async)?.username}}</button>
      <button mat-button [routerLink]="['/products']" routerLinkActive="router-link-active">Products</button>
      <button mat-button>Logout</button>
    </span>
    <span *ngIf="!(user$ | async)">
      <span mat-button [routerLink]="['/auth/login']" routerLinkActive="router-link-active">Login</span>
    </span>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```

{% endcode %}

## Extras

### 1. Convert Layout component into a pure container component 

* Add a toolbar presentational component.
* Pass user into presentational component via inputs.
