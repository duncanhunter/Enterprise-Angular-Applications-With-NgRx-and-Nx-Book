---
description: In this section we explore Strongly Typing the State and Actions
---

# 12 - Strong Typing the State and Actions

## 1. Add Login Action Creators

{% code title="libs/auth/src/lib/+state/auth.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { Authenticate, User } from '@demo-app/data-models';

export enum AuthActionTypes {
  Login = '[Auth Page] Login',
  LoginSuccess = '[Auth API] Login Success',
  LoginFail = '[Auth API] Login Fail'
}

export class Login implements Action {
  readonly type = AuthActionTypes.Login;
  constructor(public payload: Authenticate) {}
}
export class LoginSuccess implements Action {
  readonly type = AuthActionTypes.LoginSuccess;
  constructor(public payload: User) {}
}

export class LoginFail implements Action {
  readonly type = AuthActionTypes.LoginFail;
  constructor(public payload: any) {}
}

export type AuthActions = Login | LoginSuccess | LoginFail;

```
{% endcode %}

## Action Hygiene

* Actions should capture unique events not commands
* Try not to reuse actions and make generic actions
* Suffix your Action types with their source so you know where they are dispatched from like `Login = '[Login Page] Login'`
* Let effects and reducers be the decision maker not the component and add multiple cases to a switch statement or effects.
* Avoid action sub typing by adding conditional information to a property of an action payload by making multiple actions for each case. This makes it easier to test and avoids complicated conditional logic in effects and reducers.
* Write actions first

{% hint style="info" %}
Great presentation on Actions [https://www.youtube.com/watch?v=JmnsEvoy-gY&t=5s](https://www.youtube.com/watch?v=JmnsEvoy-gY&t=5s)
{% endhint %}

### 2. Add default state and interface

* Update state interface

{% code title="libs/auth/src/lib/+state/auth.reducer.ts" %}
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
  error: Error
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
{% endcode %}

## 

