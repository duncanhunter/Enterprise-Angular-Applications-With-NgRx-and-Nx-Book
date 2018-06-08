# 8 - Routing

### 1. Add a lib for a users profile page

* Add a lazy loaded lib with routing. Note this will add linting rules to .angular-cli.json to stop adding this module to other modules.

```bash
ng g lib user-profile --routing --lazy --parent-module=apps/customer-portal/src/app/app.module.ts
```

* Add a user-profile container component.

```bash
ng g c containers/user-profile -a=user-profile
```

### 2. Add a method in the subscription to navigate to the login page on login

* Run the following command to navigate

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { AuthService } from './../../services/auth/auth.service';
import { Authenticate, User } from '@demo-app/data-models';
import { Router } from '@angular/router';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor(private router: Router, private authService: AuthService) {}

  ngOnInit() {}

  login(authenticate: Authenticate) {
    this.authService
      .login(authenticate)
      .subscribe((user: User) =>
        this.router.navigate([`/user-profile/${user.id}`])
      );
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add default routes to app module

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/app/app.module.ts" %}
```typescript
RouterModule.forRoot(
    [
      { path: '', pathMatch: 'full', redirectTo: 'user-profile' },
      { path: 'auth', children: authRoutes },
      {
        path: 'user-profile',
        loadChildren: '@demo-app/user-profile#UserProfileModule'
      }
    ],
    {
       initialNavigation: 'enabled'
    }
),
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="libs/user-profile/src/user-profile.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserProfileComponent } from './containers/user-profile/user-profile.component';
import { RouterModule } from '@angular/router';

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([{ path: ':id', component: UserProfileComponent }])
  ],
  declarations: [UserProfileComponent]
})
export class UserProfileModule {}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Login again to check the routing is correctly configured.

### 3. Add a route guard to protect profile page

* Generate a guard wit the CLI

```text
ng g guard guards/auth/auth -a=auth
```

* Register services and the guard in the providers

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/auth.module.ts" %}
```typescript
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { HttpClientModule } from '@angular/common/http';
import { AuthService } from './services/auth.service';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from './guards/auth/auth.guard';
import { LoginFormComponent } from './components/login-form/login-form.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent, LoginFormComponent];

@NgModule({
  imports: [
    CommonModule,
    RouterModule,
    HttpClientModule,
    MaterialModule,
    ReactiveFormsModule
  ],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [AuthService, AuthGuard]
})
export class AuthModule {}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Update the authService to set a local flag before we add ngrx later

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/services/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs/operators';

@Injectable()
export class AuthService {
  isAuthenticated: boolean;

  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate): Observable<User> {
    return this.httpClient
      .post<User>('http://localhost:3000/login', authenticate)
      .pipe(tap(user => (this.isAuthenticated = true)));
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add auth guard logic

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/guards/auth/auth.guard.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { Observable } from 'rxjs/Observable';
import { AuthService } from './../../services/auth/auth.service';

@Injectable()
export class AuthGuard implements CanActivate {

  constructor(
    private router: Router,
    private authService: AuthService) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    if(this.authService.isAuthenticated) {
      return true;
    } else {
      this.router.navigate([`/auth/login`]);
      return false;
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add auth guard to main routes

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
import { authRoutes, AuthModule } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { AuthGuard } from '@demo-app/auth';

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
      ],
      {
        initialNavigation: 'enabled'
      }
    ),
    // Note: Issue with storeFreeze to be fixed in NgRx v6https://github.com/nrwl/nx/issues/436
    //StoreModule.forRoot({},{ metaReducers : !environment.production ? [storeFreeze] : [] }),
    StoreModule.forRoot({}),    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule,
    AuthModule,
    BrowserAnimationsModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add a temporary "debugger" to set a break point on the route guard to check it is working correctly.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/guards/auth/auth.guard.ts" %}
```typescript
  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    if(this.authService.isAuthenticated) {
      debugger;
      return true;
    } else {
      this.router.navigate([`/auth/login`]);
      return false;
    }
  }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### BONUS SECTION: Add angular interceptor

* Update auth service to set a token in local storage

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/services/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { tap } from 'rxjs/operators';
import { User, Authenticate } from '@demo-app/data-models';

@Injectable()
export class AuthService {
  user: User;
  isAuthenticated: boolean;

  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate) {
    return this.httpClient
      .post('http://localhost:3000/login', authenticate)
      .pipe(
        tap((user: User) => {
          this.isAuthenticated = true;
          this.user = user;
          localStorage.setItem('token', user.token);
        })
      );
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add an angular interceptor in a new folder in the auth lib

{% hint style="info" %}
Note: Currently there is no ng generate command for interceptors so we need to add it manually.

[https://github.com/angular/angular-cli/issues/6937](https://github.com/angular/angular-cli/issues/6937)
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/interceptors/auth/auth.interceptor.ts" %}
```typescript
import { Injectable, Injector } from '@angular/core';
import {
  HttpEvent,
  HttpInterceptor,
  HttpHandler,
  HttpRequest
} from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import { AuthService } from './../../services/auth/auth.service';
import { switchMap, tap } from 'rxjs/operators';
import { of } from 'rxjs/Observable/of';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  authService: AuthService;
  constructor(private injector: Injector) {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    this.authService = this.injector.get(AuthService);
    const token = localStorage.getItem('token');

    if (token) {
      const authReq = req.clone({
        headers: req.headers.set('Authorization', token)
      });
      return next.handle(authReq);
    } else {
      return next.handle(req);
    }
  }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title=undefined %}
```text

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Export auth interceptors from the auth module

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/auth.module.ts" %}
```typescript
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { LoginComponent } from './containers/login/login.component';
import { AuthService } from './services/auth/auth.service';
import { LoginFormComponent } from './components/login-form/login-form.component';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from './guards/auth/auth.guard';
import { AuthInterceptor } from '@demo-app/auth/src/interceptors/auth/auth.interceptor';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent, LoginFormComponent];

@NgModule({
  imports: [
    CommonModule,
    RouterModule,
    HttpClientModule,
    MaterialModule,
    ReactiveFormsModule
  ],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [
    AuthService,
    AuthGuard,
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
})
export class AuthModule {}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Check the interceptor is adding a Header

* Try and login again and look in the network traffic of the dev tools to see the Header is being added.

![Authorization header on all requests](../.gitbook/assets/image%20%283%29.png)

