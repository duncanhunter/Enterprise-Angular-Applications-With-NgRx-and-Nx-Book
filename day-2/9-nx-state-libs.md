# 9 - Nx State Libs

## 1. Add auth NgRx state

```text
ng generate ngrx auth --module=libs/auth/src/auth.module.ts
```

* Update default code generated actions.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { User, Authenticate } from '@demo-app/data-models';

export enum AuthActionTypes {
  Login = '[AuthState] Login',
  LoginSuccess = '[AuthState] Login Success',
  LoginFail = '[AuthState] Login Fail'
}

export class LoginAction implements Action {
  readonly type = AuthActionTypes.Login;
  constructor(public payload: Authenticate) {}
}

export class LoginSuccessAction implements Action {
  readonly type = AuthActionTypes.LoginSuccess;
  constructor(public payload: User) {}
}

export class LoginFailAction implements Action {
  readonly type = AuthActionTypes.LoginFail;
  constructor(public payload) {}
}

export type AuthActions = LoginAction | LoginFailAction | LoginSuccessAction;

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Action Hygiene

* Actions should capture unique events not commands
* Try not to reuse actions and make generic actions
* Suffix you Action types with their source so you know where they are dispatched from like ` Login = '[Login Page] Login'`
* Let effects and reducers be the decision maker not the component and add multiple cases to a switch statement or effects.
* Avoid action sub typing by adding conditional information to a property of an action payload by making multiple actions for each case. This makes it easier to test and avoids complicated conditional logic in effects and reducers.
* Write actions first

{% hint style="info" %}
Great presentation on Actions [https://www.youtube.com/watch?v=JmnsEvoy-gY&t=5s](https://www.youtube.com/watch?v=JmnsEvoy-gY&t=5s)
{% endhint %}

### 2. Add default state and interface

* Update state interface

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.reducer.ts" %}
```typescript
//----- ABBREVIATED-----//


/**
 * Interface for the 'Auth' data used in
 *  - AuthState, and
 *  - authReducer
 */
export interface AuthData {
  loading: boolean;
  user: User;
}

/**
 * Interface to the part of the Store containing AuthState
 * and other information related to AuthData.
 */
export interface AuthState {
  readonly auth: AuthData;
}

//----- ABBREVIATED-----//
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Add an effect

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect } from '@ngrx/effects';
import * as authActions from './auth.actions';
import { AuthState } from './auth.reducer';
import { DataPersistence } from '@nrwl/nx';
import { AuthActionTypes } from './auth.actions';
import { AuthService } from '@demo-app/auth/src/services/auth/auth.service';
import { map } from 'rxjs/operators';
import { User } from '@demo-app/data-models';

@Injectable()
export class AuthEffects {

  @Effect()
  loadAuth$ = this.dataPersistence.fetch(AuthActionTypes.Login, {
    run: (action: authActions.LoginAction) => {
      return this.authService.login(action.payload).pipe(map((user: User) => (new authActions.LoginSuccessAction(user))))
    },

    onError: (action: authActions.LoginAction, error) => {
      return new authActions.LoginFailAction(error);
    }
  });

  constructor(
    private actions$: Actions,
    private dataPersistence: DataPersistence<AuthState>,
    private authService: AuthService
  ) {}
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 4. Add reducer code

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.reducer.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { AuthActions, AuthActionTypes } from './auth.actions';
import { User } from '@demo-app/data-models';

/**
 * Interface for the 'Auth' data used in
 *  - AuthState, and
 *  - authReducer
 */
export interface AuthData {
  loading: boolean;
  user: User;
}

/**
 * Interface to the part of the Store containing AuthState
 * and other information related to AuthData.
 */
export interface AuthState {
  readonly auth: AuthData;
}

export const initialState: AuthData = {
  loading: false,
  user: null
};

export function authReducer(
  state = initialState,
  action: AuthActions
): AuthData {
  switch (action.type) {
    case AuthActionTypes.Login:
      return {
        ...state,
        loading: true
      }

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
{% endcode-tabs-item %}
{% endcode-tabs %}

### 5. Update LoginComponent to dispatch action

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit, ChangeDetectionStrategy, } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { AuthService } from './../../services/auth/auth.service';
import { Router } from '@angular/router';
import { AuthState } from './../../+state/auth.reducer';
import { Store } from '@ngrx/store';
import * as authActions from './../../+state/auth.actions';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class LoginComponent {

  constructor(
    private router: Router,
    private store: Store<AuthState>) { }

  login(authenticate: Authenticate) {
   this.store.dispatch(new authActions.LoginAction(authenticate));
  }

}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 6. Add new Effect action to navigate on LoginSuccess

* Add new effect to manage routing

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.actions.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import * as authActions from './auth.actions';
import { AuthState } from './auth.reducer';
import { DataPersistence } from '@nrwl/nx';
import { AuthActionTypes } from './auth.actions';
import { AuthService } from '@demo-app/auth/src/services/auth/auth.service';
import { map, tap } from 'rxjs/operators';
import { User } from '@demo-app/data-models';
import { Router } from '@angular/router';

@Injectable()
export class AuthEffects {

  @Effect()
  loadAuth$ = this.dataPersistence.fetch(AuthActionTypes.Login, {
    run: (action: authActions.LoginAction) => {
      return this.authService.login(action.payload).pipe(map((user: User) => (new authActions.LoginSuccessAction(user))))
    },

    onError: (action: authActions.LoginAction, error) => {
      return new authActions.LoginFailAction(error);
    }
  });

  @Effect({dispatch: false})
  navigateToProfile$ = this.actions$.pipe(
    ofType(AuthActionTypes.LoginSuccess),
    map((action: authActions.LoginSuccessAction) => action.payload),
    tap((user: User) => this.router.navigate([`/user-profile/${user.id}`]))
  )

  constructor(
    private actions$: Actions,
    private dataPersistence: DataPersistence<AuthState>,
    private authService: AuthService,
    private router: Router
  ) {}
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 7. Export state references in index.ts

{% code-tabs %}
{% code-tabs-item title="libs/auth/index.ts" %}
```typescript
export { AuthModule, authRoutes } from './src/auth.module';
export { AuthGuard } from './src/guards/auth/auth.guard';
export { AuthState } from './src/+state/auth.reducer';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

