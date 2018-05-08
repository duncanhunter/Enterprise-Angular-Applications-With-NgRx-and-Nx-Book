# 9 - Nx state libs

### 1. Add the NgRx schematics

```text
npm install @ngrx/schematics@5.2.0 --save-dev
```

{% hint style="info" %}
Note: You can learn more here [https://github.com/ngrx/platform/blob/master/docs/schematics/README.md](https://github.com/ngrx/platform/blob/master/docs/schematics/README.md)
{% endhint %}

### 2. Add auth NgRx state

```text
ng generate ngrx auth --module=libs/auth/src/auth.module.ts
```

* Delete the actions file from the nx code generated files at the path _**`libs/auth/src/+state/auth.actions.ts`**_

{% code-tabs %}
{% code-tabs-item title="osx" %}
```text
rm -rf libs/auth/src/+state/auth.actions.ts
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="windows" %}
```text
del /f libs/auth/src/+state/auth.actions.ts
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Run NgRx schematic generate command for actions

```text
ng generate action +state/auth -a=auth  --collection @ngrx/schematics
```

* Swap out original for new nx style effect

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { User, Authenticate } from '@demo-app/data-models';

export enum AuthStateActionTypes {
Login = '[AuthState] Login',
LoginSuccess = '[AuthState] Login Success',
LoginFail = '[AuthState] Login Fail'
}

export class LoginAction implements Action {
readonly type = AuthStateActionTypes.Login;
constructor(public payload: Authenticate) {}
}

export class LoginSuccessAction implements Action {
readonly type = AuthStateActionTypes.LoginSuccess;
constructor(public payload: User) {}
}

export class LoginFailAction implements Action {
readonly type = AuthStateActionTypes.LoginFail;
constructor(public payload) {}
}

export type AuthStateActions =
LoginAction
| LoginFailAction
| LoginSuccessAction;
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

### Add default state and interface

* Update state interface

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.reducer.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { AuthStateActions, AuthStateActionTypes } from './auth.actions';
import { User } from '@demo-app/data-models';

export interface AuthData {
  user: User,
  loading: boolean
}

export interface AuthState {
  readonly auth: AuthData;
}

export const initialState: AuthData = {
  user: null,
  loading: false
};
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Add an effect

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { of } from 'rxjs/observable/of';
import { AuthData } from './auth.reducer';
import * as authActions from './auth.actions';
import { map, catchError, tap, mergeMap } from 'rxjs/operators';
import { AuthService } from './../services/auth.service';

@Injectable()
export class AuthEffects {
  @Effect()
  login = this.actions$
    .ofType(authActions.AuthStateActionTypes.Login)
    .pipe(
      mergeMap((action: authActions.LoginAction) =>
        this.authService
          .login(action.payload)
          .pipe(
            map((user: User) => new authActions.LoginSuccessAction(user)),
            catchError(error => of(new authActions.LoginFailAction(error)))
          )
      )
    );

  constructor(private actions$: Actions, private authService: AuthService) {}
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Swap out original for new nx style effect

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { of } from 'rxjs/observable/of';
import * as authActions from './auth.actions';
import { map, catchError, tap, mergeMap } from 'rxjs/operators';
import { AuthService } from './../services/auth/auth.service';
import { DataPersistence } from '@nrwl/nx';
import { AuthData } from './auth.reducer';

@Injectable()
export class AuthEffects {
  @Effect()
  login$ = this.dataPersistence.fetch(authActions.AuthStateActionTypes.Login, {
    run: (action: authActions.LoginAction, state: AuthData) => {
      return this.authService
        .login(action.payload)
        .pipe(map(user => new authActions.LoginSuccessAction(user)));
    },

    onError: (action: authActions.LoginAction, error) => {
      return of(new authActions.LoginFailAction(error));
    }
  });

  constructor(
    private actions: Actions,
    private dataPersistence: DataPersistence<AuthData>,
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
import { AuthStateActions, AuthStateActionTypes } from './auth.actions';
import { User } from '@demo-app/data-models';

export interface AuthData {
  user: User,
  loading: boolean
}

export interface AuthState {
  readonly auth: AuthData;
}

export const initialState: AuthData = {
  user: null,
  loading: false
};

export function authReducer(
  state: AuthData,
  action: AuthStateActions
): AuthData {
  switch (action.type) {
    case AuthStateActionTypes.Login: {
      return {
        ...state,
        loading: true
      };
    }
    case AuthStateActionTypes.LoginSuccess: {
      return {
        ...state,
        loading: false,
        user: action.payload
      };
    }
    default: {
      return state;
    }
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 5. Update LoginComponent to dispatch action

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormControl, Validators } from '@angular/forms';
import { User, Authenticate } from '@demo-app/data-models';
import { Store } from '@ngrx/store';
import * as authActions from './../../+state/auth.actions';
import { AuthData } from '@demo-app/auth/src/+state/auth.reducer';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor(private store: Store<AuthData>) {}

  ngOnInit() {}

  login(authenticate: Authenticate): void {
    this.store.dispatch(new authActions.LoginAction(authenticate));
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 6. Add route change action on success

* Add a new action to navigate

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { User, Authenticate } from '@demo-app/data-models';

export enum AuthStateActionTypes {
Login = '[AuthState] Login',
LoginSuccess = '[AuthState] Login Success',
LoginFail = '[AuthState] Login Fail',
NavigateToProfile = '[AuthState] Navigate To Profile'
}

export class LoginAction implements Action {
readonly type = AuthStateActionTypes.Login;
constructor(public payload: Authenticate) {}
}

export class LoginSuccessAction implements Action {
readonly type = AuthStateActionTypes.LoginSuccess;
constructor(public payload: User) {}
}

export class LoginFailAction implements Action {
readonly type = AuthStateActionTypes.LoginFail;
constructor(public payload) {}
}

export class NavigateToProfileAction implements Action {
readonly type = AuthStateActionTypes.NavigateToProfile;
constructor(public payload:number) {}
}

export type AuthStateActions =
LoginAction
| LoginFailAction
| LoginSuccessAction
| NavigateToProfileAction;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add new Effect action to navigate

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { of } from 'rxjs/observable/of';
import 'rxjs/add/operator/switchMap';
import * as authActions from './auth.actions';
import { map, catchError, tap, mergeMap } from 'rxjs/operators';
import { AuthService } from './../services/auth/auth.service';
import { DataPersistence } from '@nrwl/nx';
import { Router } from '@angular/router';
import { User } from '@demo-app/data-models';
import { AuthData } from './auth.reducer';

@Injectable()
export class AuthEffects {
  @Effect()
  login$ = this.dataPersistence.fetch(authActions.AuthStateActionTypes.Login, {
    run: (action: authActions.LoginAction, state: AuthData) => {
      return this.authService
        .login(action.payload)
        .pipe(
          mergeMap((user: User) => [
            new authActions.LoginSuccessAction(user),
            new authActions.NavigateToProfileAction(user.id)
          ])
        );
    },

    onError: (action: authActions.LoginAction, error) => {
      return of(new authActions.LoginFailAction(error));
    }
  });

  @Effect({ dispatch: false })
  navigateToProfile = this.actions
    .ofType(authActions.AuthStateActionTypes.NavigateToProfile)
    .pipe(
      map((action: authActions.NavigateToProfileAction) =>
        this.router.navigate([`/user-profile/${action.payload}`])
      )
    );

  constructor(
    private actions: Actions,
    private dataPersistence: DataPersistence<AuthData>,
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
export { AuthData } from './src/+state/auth.reducer';

// export * from './src/+state';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

