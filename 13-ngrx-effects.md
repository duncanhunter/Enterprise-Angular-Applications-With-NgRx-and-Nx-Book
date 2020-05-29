---
description: In this section we examine adding effects
---

# 13 - NgRx Effects

## 1. Add an Auth Effects

{% code title="libs/auth/src/lib/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { AuthActionTypes } from './auth.actions';
import { mergeMap, map, catchError, tap } from 'rxjs/operators';
import { of } from 'rxjs';
import { AuthService } from './../services/auth/auth.service';
import * as authActions from './auth.actions';
import { User } from '@demo-app/data-models';

@Injectable()
export class AuthEffects {
  @Effect()
  login$ = this.actions$.pipe(
    ofType(AuthActionTypes.Login),
    mergeMap((action: authActions.Login) =>
      this.authService
        .login(action.payload)
        .pipe(
          map((user: User) => new authActions.LoginSuccess(user)),
          catchError(error => of(new authActions.LoginFail(error)))
        )
    )
  );

  constructor(
    private actions$: Actions,
    private authService: AuthService
  ) {}
}

```
{% endcode %}

## 2. Add reducer code

{% code title="libs/auth/src/lib/+state/auth.reducer.ts" %}
```typescript
import { AuthActions, AuthActionTypes } from './auth.actions';
import { User } from '@demo-app/data-models';

export interface AuthData {
  loading: boolean;
  user: User;
  error: '';
}
export interface AuthState {
  readonly auth: AuthData;
}

export const initialState: AuthData = {
  error: '',
  user: null,
  loading: false
};

export function authReducer(
  state = initialState,
  action: AuthActions
): AuthData {
  switch (action.type) {
    case AuthActionTypes.Login:
      return { ...state, loading: true };

    case AuthActionTypes.LoginSuccess: {
      return { ...state, user: action.payload, loading: false };
    }
    case AuthActionTypes.LoginFail: {
      return { ...state, user: null, loading: false };
    }
    default:
      return state;
  }
}

```
{% endcode %}

## 3. Update LoginComponent to dispatch an action

{% code title="libs/auth/src/lib/containers/login/login.component.ts" %}
```typescript
import { Component, ChangeDetectionStrategy, } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { AuthState } from './../../+state/auth.reducer';
import { Store } from '@ngrx/store';
import * as authActions from './../../+state/auth.actions';
​
@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class LoginComponent {
​
  constructor(
    private store: Store<AuthState>) { }
​
  login(authenticate: Authenticate) {
   this.store.dispatch(new authActions.Login(authenticate));
  }
​
}
```
{% endcode %}

## 4. Add new Effect action to navigate on LoginSuccess

* Add new effect to manage routing

You can read more about routing with actions here [https://github.com/ngrx/platform/blob/master/docs/router-store/api.md\#navigation-actions](https://github.com/ngrx/platform/blob/master/docs/router-store/api.md#navigation-actions).

{% code title="libs/auth/src/lib/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { AuthActionTypes } from './auth.actions';
import { mergeMap, map, catchError, tap } from 'rxjs/operators';
import { of } from 'rxjs';
import { AuthService } from './../services/auth/auth.service';
import * as authActions from './auth.actions';
import { User } from '@demo-app/data-models';
import { Router } from '@angular/router';

@Injectable()
export class AuthEffects {
  @Effect()
  login$ = this.actions$.pipe(
    ofType(AuthActionTypes.Login),
    mergeMap((action: authActions.Login) =>
      this.authService
        .login(action.payload)
        .pipe(
          map((user: User) => new authActions.LoginSuccess(user)),
          catchError(error => of(new authActions.LoginFail(error)))
        )
    )
  );

  @Effect({ dispatch: false })
  navigateToProfile$ = this.actions$.pipe(
    ofType(AuthActionTypes.LoginSuccess),
    map((action: authActions.LoginSuccess) => action.payload),
    tap(() => this.router.navigate([`/products`]))
  );

  constructor(
    private actions$: Actions,
    private authService: AuthService,
    private router: Router
  ) {}
}

```
{% endcode %}

## 5. Export state references in index.ts

{% code title="libs/auth/index.ts" %}
```typescript
export * from './lib/auth.module';
export { AuthService } from './lib/services/auth/auth.service';
export { AuthGuard } from './lib/guards/auth/auth.guard';
export { AuthState } from './lib/+state/auth.reducer';

```
{% endcode %}

### 6. Update AuthGuard to use the store

```typescript
import { Injectable } from '@angular/core';
import {
  CanActivate,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  Router
} from '@angular/router';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { AuthState } from '@demo-app/auth';
import { Store, select } from '@ngrx/store';
​
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(private router: Router, private store: Store<AuthState>) { }
​
  canActivate(
    next: ActivatedRouteSnapshot,
    routerState: RouterStateSnapshot
  ): Observable<boolean> {
    return this.store.pipe(select((state) => state.auth.user),
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



## 7. On load check local storage and dispatch a LoginSuccess action

{% code title="apps/customer-portal/src/app/app.component.ts" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState } from '@demo-app/auth';
import { LoginSuccess } from '@demo-app/auth/src/lib/+state/auth.actions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'app';

  constructor(private store: Store<AuthState>){
      const user = JSON.parse(localStorage.getItem('user'));
      if(user) {
        this.store.dispatch(new LoginSuccess(user))
      }
    };

}

```
{% endcode %}

