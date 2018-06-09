# 11 - Adding NgRx to Nx App

#### 5. Add ngrx

* Add a default set up for ngrx to our new app. We can run the generate command for ngrx with the module and --onlyEmptyRoot option to only add the StoreModule.forRoot and EffectsModule.forRoot calls without generating any new files versus --root which will make a default reducer and effect.

```
ng g ngrx app --module=apps/customer-portal/src/app/app.module.ts  --onlyEmptyRoot

## NgRx schematics

* Ngrx schematics have a different syntax to write effects versus using Nrwl's DataPersitance library which you need to know to understand the normal way to do it. The schematics from the NgRx team can be added with the below code.

```text
npm install @ngrx/schematics@5.2.0 --save-dev
```

{% hint style="info" %}
Note: You can learn more here [https://github.com/ngrx/platform/blob/master/docs/schematics/README.md](https://github.com/ngrx/platform/blob/master/docs/schematics/README.md)
{% endhint %}

## 1. Add NgRx based effect syntax

* Comment out the existing effect and use the normal effect syntax.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/+state/auth.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import * as authActions from './auth.actions';
import { AuthState } from './auth.reducer';
import { DataPersistence } from '@nrwl/nx';
import { AuthActionTypes } from './auth.actions';
import { AuthService } from '@demo-app/auth/src/services/auth/auth.service';
import { map, tap, mergeMap, catchError } from 'rxjs/operators';
import { User } from '@demo-app/data-models';
import { Router } from '@angular/router';
import { of } from 'rxjs/observable/of';

@Injectable()
export class AuthEffects {

  // @Effect()
  // loadAuth$ = this.dataPersistence.fetch(AuthActionTypes.Login, {
  //   run: (action: authActions.LoginAction) => {
  //     return this.authService.login(action.payload).pipe(map((user: User) => (new authActions.LoginSuccessAction(user))))
  //   },

  //   onError: (action: authActions.LoginAction, error) => {
  //     return new authActions.LoginFailAction(error);
  //   }
  // });

  @Effect()
  login = this.actions$
    .ofType(AuthActionTypes.Login)
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

