# 9 - Route Guards and Products Lib

## 1. Add a lib for a products page

* Add a lazy loaded lib with routing. We will grow this new feature lib out with advance NgRx features later int he course but for now we will just make the container component to navigate to on login.

```bash
nx generate @nrwl/angular:lib products --routing --lazy --parent-module=apps/customer-portal/src/app/app.module.ts
...
CREATE libs/products/README.md
CREATE libs/products/tsconfig.lib.json
CREATE libs/products/src/index.ts
CREATE libs/products/src/lib/products.module.ts
CREATE libs/products/tsconfig.json
CREATE libs/products/jest.config.js
CREATE libs/products/src/test-setup.ts
CREATE libs/products/tsconfig.spec.json
CREATE libs/products/.eslintrc.json
UPDATE package.json
UPDATE workspace.json
UPDATE nx.json
UPDATE tsconfig.base.json
UPDATE .vscode/extensions.json
UPDATE jest.config.js
UPDATE apps/customer-portal/src/app/app.module.ts
UPDATE apps/customer-portal/tsconfig.app.json
```

* Add a products container component.

```bash
nx g @nrwl/angular:component  containers/products --project=products
CREATE libs/products/src/lib/containers/products/products.component.html (23 bytes)
CREATE libs/products/src/lib/containers/products/products.component.spec.ts (642 bytes)
CREATE libs/products/src/lib/containers/products/products.component.ts (286 bytes)
CREATE libs/products/src/lib/containers/products/products.component.css (0 bytes)
UPDATE libs/products/src/lib/products.module.ts (449 bytes)
```

## 2. A default app route to always go to products page

* Check default routes added to AppModule
* Add a new default route to always load the products on app load

{% code title="apps/customer-portal/src/app/app.module.ts" %}

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations'; // Added
import { LayoutModule } from '@demo-app/layout';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    NxModule.forRoot(),
    RouterModule.forRoot(
      [
        { path: '', pathMatch: 'full', redirectTo: 'products' }, // added
        { path: 'auth', children: authRoutes },
        {
          path: 'products',
          loadChildren: () =>
            import('@demo-app/products').then(
              (mod) => mod.ProductsModule                       // added
            ),
        },
      ],
      {
        initialNavigation: 'enabled',
      }
    ),
    AuthModule,
    LayoutModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

{% endcode %}

## 3. Add ProductsModule route

{% code title="libs/products/src/lib/products.module.ts" %}

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { ProductsComponent } from './containers/products/products.component';

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([{ path: '', component: ProductsComponent }]),
  ],
  declarations: [ProductsComponent],
})
export class ProductsModule {}
```

{% endcode %}

* Login again at the '/auth/login' route to check the routing is correctly configured by trying to click the 'products' button in the main menu.

## 4. Add a route guard to protect products page

* Generate a guard with the CLI

```text
nx g @nrwl/angular:guard  guards/auth/auth --project=auth
```

The CLI will ask you what functions to implement.  We will only be needing CanActivate for now, so just press the ENTER key.

```bash
? Which interfaces would you like to implement? (Press <space> to select, <a> to toggle
 all, <i> to invert selection)CanActivate
CREATE libs/auth/src/lib/guards/auth/auth.guard.spec.ts (331 bytes)
CREATE libs/auth/src/lib/guards/auth/auth.guard.ts (456 bytes)
```

* Add auth guard logic

{% code title="libs/auth/src/lib/guards/auth/auth.guard.ts" %}

```typescript
import { Injectable } from '@angular/core';
import {
  CanActivate,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  Router
} from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from '../../services/auth/auth.service';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(private router: Router, private authService: AuthService) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> {
    return this.authService.user$.pipe(
      map(user => {
        if (user) {
          return true;
        } else {
          this.router.navigate([`/auth/login`]);
          return false;
        }
      })
    );
  }
}

```

{% endcode %}

## 5. Add auth guard to main routes

* Re-export the guard from the index.ts file in the Auth module.

{% code title="libs/auth/src/index.ts" %}

```typescript
export * from './lib/auth.module';
export { AuthService } from './lib/services/auth/auth.service';
export { AuthGuard } from './lib/guards/auth/auth.guard';
```

{% endcode %}

* Apply the guard to the Customer Portal main routes.

{% code title="apps/customer-portal/src/app/app.module.ts" %}

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { LayoutModule } from '@demo-app/layout';
import { AuthGuard } from '@demo-app/auth'; // added

@NgModule({
  declarations: [AppComponent],
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
          loadChildren: () =>
            import('@demo-app/products').then(
              (module) => module.ProductsModule
            ),
          canActivate: [AuthGuard],      // added
        },
      ],
      {
        initialNavigation: 'enabled',
      }
    ),
    AuthModule,
    LayoutModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

{% endcode %}

Notice the syntax for lazy loading. It's shown in [the official docs])https://angular.io/guide/lazy-loading-ngmodules) as the --module option.

The docs also say it uses *loadChildren followed by a function that uses the browser's built-in import('...') syntax for dynamic imports. The import path is the relative path to the module.*

## 6. Check the route guard is working

* Add a temporary "debugger" to set a break point on the route guard to check it is working correctly.

{% code title="libs/auth/src/lib/guards/auth/auth.guard.ts" %}

```typescript
/// abbreviated
canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> {
    return this.authService.user$.pipe(
      map(user => {
        if (user) {
          // eslint-disable-next-line no-debugger
          debugger;
          return true;
        } else {
          this.router.navigate([`/auth/login`]);
          return false;
        }
      })
    );
  }
/// abbreviated  
```

{% endcode %}

## 7.  Cache the user in local storage to save logging in for the rest or the workshop

* Load BehaviorSubject on Service creation with a user from local storage if one exists.

{% code title="libs/auth/src/lib/services/auth/auth.service.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { tap } from 'rxjs/operators';
​
@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private userSubject$ = new BehaviorSubject<User>(null);
  user$ = this.userSubject$.asObservable();
​
  constructor(private httpClient: HttpClient) {
    const user = localStorage.getItem('user');
    if (user) {
      this.userSubject$.next(JSON.parse(user));
    }
  }
​
  login(authenticate: Authenticate): Observable<User> {
    return this.httpClient.post<User>(
      'http://localhost:3000/login',
      authenticate
    ).pipe(tap((user: User) => {
        this.userSubject$.next(user);
        localStorage.setItem('user', JSON.stringify(user));
    }));
  }
​
}
```

{% endcode %}

![How to delete local storage](.gitbook/assets/image%20%2822%29.png)

## Extras

### 1. Add logout functionality

Try these steps below to logout a user.

* Add logout button to main menu
* Call service and logout the user by clearing the behaviour subject
* Navigate to login page

### 2. Add angular interceptor

### a\) Update auth service to set a token in local storage

* Add an angular interceptor in a new folder in the auth lib

{% hint style="info" %}
Note: Currently there is no ng generate command for interceptors so we need to add it manually.

[https://github.com/angular/angular-cli/issues/6937](https://github.com/angular/angular-cli/issues/6937)
{% endhint %}

{% tabs %}
{% tab title="libs/auth/src/lib/interceptors/auth/auth.interceptor.ts" %}

```typescript
import { Injectable, Injector } from '@angular/core';
import {
  HttpEvent,
  HttpInterceptor,
  HttpHandler,
  HttpRequest
} from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import { switchMap, tap } from 'rxjs/operators';
import { of } from 'rxjs/Observable/of';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
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

{% endtab %}

{% tab title="Plain Text" %}

```text

```

{% endtab %}
{% endtabs %}

* Export auth interceptors from the auth module

{% code title="libs/auth/src/lib/auth.module.ts" %}

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
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
})
export class AuthModule {}
```

{% endcode %}

### b\) Check the interceptor is adding a Header

* Try and login again and look in the network traffic of the dev tools to see the Header is being added.

![Authorization header on all requests](.gitbook/assets/image%20%284%29.png)
