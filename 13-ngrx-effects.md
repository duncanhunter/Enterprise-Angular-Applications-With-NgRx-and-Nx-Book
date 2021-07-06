---
description: In this section we examine adding effects
---

# 13 - NgRx Effects

[The official docs](https://ngrx.io/guide/effects) say *Effects decrease the responsibility of the component.*

## 1. Add an Auth Effects

{% code title="libs/auth/src/lib/+state/auth.effects.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { createEffect, Actions, ofType } from '@ngrx/effects';
import { fetch } from '@nrwl/angular';
import { AuthActionTypes } from './auth.actions';
import * as AuthActions from './auth.actions';
import { AuthService } from './../services/auth/auth.service';

@Injectable()
export class AuthEffects {
  login$ = createEffect(() => this.actions$.pipe(
    ofType(AuthActionTypes.Login),
    fetch({
      run: (action) => {
        return this.authService.login(action);
      },
      onError: (action, error) => {
        return AuthActions.loginFailure(error);
      },
    })
  ));

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
import { createReducer, on, Action } from '@ngrx/store';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import * as AuthActions from './auth.actions';
import { AuthEntity } from './auth.models';
import { User } from '@demo-app/data-models';

export const AUTH_FEATURE_KEY = 'auth';

export interface AuthData {
  loading: boolean;
  user: User;
  error: Error;
}

export interface AuthState {
  readonly auth: AuthData;
}

export interface State extends EntityState<AuthEntity> {
  selectedId?: string | number;
  loaded: boolean;
  error?: string | null;
}

export interface AuthPartialState {
  readonly [AUTH_FEATURE_KEY]: State;
}

export const authAdapter: EntityAdapter<AuthEntity> = createEntityAdapter<AuthEntity>();

export const initialState: State = authAdapter.getInitialState({
  action: AuthActions,
  loaded: false,
});
const authReducer = createReducer(
  initialState,
  on(AuthActions.login, (state) => ({ ...state, loading: true })),
  on(AuthActions.loginSuccess, (state) => ({
    ...state,
    user: AuthActions.loginSuccess,
    loading: false,
  })),
  on(AuthActions.loginFailure, (state) => ({
    ...state,
    user: null,
    loading: false,
  }))
);

export function reducer(state: State | undefined, action: Action) {
  return authReducer(state, action);
}
```

{% endcode %}

## 3. Update LoginComponent to dispatch an action

{% code title="libs/auth/src/lib/containers/login/login.component.ts" %}

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { AuthState } from './../../+state/auth.reducer';
import { Store } from '@ngrx/store';
import * as authActions from './../../+state/auth.actions';
@Component({
  selector: 'demo-app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class LoginComponent {

  constructor(private store: Store<AuthState>) {}
  login(authenticate: Authenticate) {
    this.store.dispatch(authActions.login({ payload: authenticate }));
  }
}
```

{% endcode %}

## 4. Add new Effect action to navigate on LoginSuccess

* Add new effect to manage routing

You can read more about routing with actions here: [Router Actions](https://ngrx.io/guide/router-store/actions).

{% code title="libs/auth/src/lib/+state/auth.effects.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { Router } from '@angular/router';
import { createEffect, Actions, ofType } from '@ngrx/effects';
import { fetch } from '@nrwl/angular';
import { map, tap } from 'rxjs/operators';
import { AuthActionTypes } from './auth.actions';
import * as AuthActions from './auth.actions';
import { AuthService } from './../services/auth/auth.service';

@Injectable()
export class AuthEffects {
  login$ = createEffect(() =>
    this.actions$.pipe(
      ofType(AuthActionTypes.Login),
      fetch({
        run: (action) => {
          return this.authService.login(action);
        },
        onError: (action, error) => {
          return AuthActions.loginFailure(error);
        },
      })
    )
  );

  navigateToProfile$ = createEffect(
    () =>
      this.actions$.pipe(
        ofType(AuthActionTypes.LoginSuccess),
        map((action: AuthActionTypes.LoginSuccess) => action),
        tap(() => this.router.navigate([`/products`]))
      ),
    { dispatch: false }
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
export * from './lib/+state/auth.actions';
export * from './lib/+state/auth.reducer';
export * from './lib/+state/auth.selectors';
export * from './lib/+state/auth.effects';
export * from './lib/+state/auth.models';
export * from './lib/auth.module';
export { AuthService } from './lib/services/auth/auth.service';
export { AuthGuard } from './lib/guards/auth/auth.guard';
export { AuthState } from './lib/+state/auth.reducer';
```

{% endcode %}

### 6. Update AuthGuard to use the store

{% code title="libs/auth/src/lib/guards/auth/auth.guard.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { AuthState } from '../../+state/auth.reducer';
import { Store, select } from '@ngrx/store';
@Injectable({
  providedIn: 'root',
})
export class AuthGuard implements CanActivate {
  constructor(private router: Router, private store: Store<AuthState>) {}
  canActivate(): Observable<boolean> {
    return this.store.pipe(
      select((state) => state.auth.user),
      map((user) => {
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
import * as AuthActions from '@demo-app/auth';

@Component({
  selector: 'demo-app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent {
  title = 'app';

  constructor(private store: Store<AuthState>) {
    const user = JSON.parse(localStorage.getItem('user'));
    if (user) {
      this.store.dispatch(AuthActions.loginSuccess(user));
    }
  }
}
```

{% endcode %}
