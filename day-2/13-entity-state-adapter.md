# 13 - Entity State Adapter

## 1. Add a new route guard

```text
ng g guard guards/auth-admin/auth-admin -a=auth
```

* check in the route guard if the person is an admin

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/guards/auth-admin/auth-admin.guard.ts" %}
```typescript
import { Injectable } from '@angular/core';
import {
  CanActivate,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  Router
} from '@angular/router';
import { Observable } from 'rxjs/Observable';
import { AuthService } from './../../services/auth/auth.service';
import { getUser } from './../../+state/index';
import { AuthState } from './../../+state/auth.reducer';
import { Store } from '@ngrx/store';
import { map } from 'rxjs/operators';
import { User } from '@demo-app/data-models';

@Injectable()
export class AuthAdminGuard implements CanActivate {
  constructor(private router: Router, private store: Store<AuthState>) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> {
    return this.store.select(getUser).pipe(
      map((user: User) => {
        if (user && user.role === 'admin') {
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
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the new Guard to the index.ts file for the module

{% code-tabs %}
{% code-tabs-item title="libs/auth/index.ts" %}
```text
export { AuthAdminGuard } from './src/guards/auth-admin/auth-admin.guard';
export { AuthGuard } from './src/guards/auth/auth.guard';
export { AuthModule, authRoutes } from './src/auth.module';
export { AuthState } from './src/+state/auth.reducer';
export * from './src/+state';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 2. Make a users lib in an admin directory

```text
ng g lib users --directory=admin-portal --routing --lazy --parent-module=apps/admin-portal/src/app/app.module.ts
```

* Add a UserList container component

```text
ng g c containers/user-list -a=admin-portal/users
```

* Add ngrx feature state to the new lib

```text
ng g ngrx users --module=libs/admin-portal/users/src/users.module.ts
```

* Add UserList component to the routes for this lib

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/users.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { UserListComponent } from './containers/user-list/user-list.component';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import {
  usersReducer,
  initialState as usersInitialState
} from './+state/users.reducer';
import { UsersEffects } from './+state/users.effects';

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([
      { path: '', pathMatch: 'full', component: UserListComponent }
    ]),
    StoreModule.forFeature('users', usersReducer, {
      initialState: usersInitialState
    }),
    EffectsModule.forFeature([UsersEffects])
  ],
  declarations: [UserListComponent],
  providers: [UsersEffects]
})
export class UsersModule {}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the route also to the admin-portal app

{% code-tabs %}
{% code-tabs-item title="apps/admin-portal/src/app/app.module.ts" %}
```typescript
 RouterModule.forRoot([
  { path: '', pathMatch: 'full', redirectTo: 'user-profile' },
  { path: 'auth', children: authRoutes },
  {
    path: 'user-profile',
    loadChildren: '@demo-app/user-profile#UserProfileModule',
    canActivate: [AuthGuard]
  },
  {
    path: 'users',
    loadChildren: '@demo-app/admin-portal/users#UsersModule',
    canActivate: [AuthAdminGuard]
  }
]),
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Update auth module to use new guard

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/auth.module.ts" %}
```typescript
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './container/login/login.component';
import { LoginFormComponent } from './components/login-form/login-form.component';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthService } from './services/auth.service';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';
import { AuthGuard } from './guards/auth.guard';
import { AuthInterceptor } from './interceptors/auth.interceptor';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { authReducer } from './+state/auth.reducer';
import { authInitialState } from './+state/auth.init';
import { AuthEffects } from './+state/auth.effects';
import { AuthAdminGuard } from './guards/auth-admin/auth-admin.guard';

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
    ReactiveFormsModule,
    StoreModule.forFeature('auth', authReducer, {initialState: authInitialState}),
    EffectsModule.forFeature([AuthEffects])
  ],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [
    AuthService,
    AuthGuard,
    AuthAdminGuard,
    AuthEffects,
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

* Update layout lib to show users menu button in an admin

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/layout/src/containers/layout/layout.component.html" %}
```markup
<mat-toolbar color="primary" fxLayout="row">
  <span>Admin Portal</span>
  <div class="right-nav">
    <span>{{(user$ | async)?.username}}</span>
    <button mat-button *ngIf="(user$ | async)?.role === 'admin'" [routerLink]="['/users']">Users</button>
  </div>
</mat-toolbar>
<ng-content></ng-content>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Make sure the LayoutModule has imported the RouterModule to use a routerlink

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/layout/src/layout.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { LayoutComponent } from './containers/layout/layout.component';
import { MaterialModule } from '@demo-app/material';
import { RouterModule } from '@angular/router';

const COMPONENTS = [LayoutComponent];

@NgModule({
  imports: [CommonModule, MaterialModule, RouterModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS]
})
export class LayoutModule {}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Make a new service for users

```text
ng g service services/users -a=admin-portal/users
```

* add a new method to the service to get users

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/services/users.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()

export class UsersService {

  constructor(private httpClient: HttpClient) { }

  getUsers() {
    return this.httpClient.get(`http://localhost:3000/users`);
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Configure NgRx state for users

* Update the actions for getting users

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/users.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { User } from '@demo-app/data-models';

export enum UsersActionTypes {

  LoadUsers = '[Users] Load',
  LoadUsersSuccess = '[Users] Load Success',
  LoadUsersFail = '[Users] Load Fail'
}

export class LoadUsersAction implements Action {
  readonly type = UsersActionTypes.LoadUsers;
}

export class LoadUsersSuccessAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(public payload: User[]) {}
}

export class LoadUsersFailAction implements Action {
  readonly type = UsersActionTypes.LoadUsersFail;
  constructor(public payload: any) {}
}

export type UsersActions =
  | LoadUsersAction
  | LoadUsersSuccessAction
  | LoadUsersFailAction;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Update the default effect logic

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/users.effects.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Effect, Actions } from '@ngrx/effects';
import { DataPersistence } from '@nrwl/nx';
import { of } from 'rxjs/observable/of'
;
import 'rxjs/add/operator/switchMap';
import { UsersState } from './users.interfaces';
import * as usersActions from './users.actions';
import { UsersService } from './../services/users.service';
import { map } from 'rxjs/operators';
import { User } from '@demo-app/data-models';

@Injectable()
export class UsersEffects {
  @Effect()
  loadData = this.dataPersistence.fetch(
    usersActions.UsersActionTypes.LoadUsers,
    {
      run: (action: usersActions.LoadUsersAction, state: UsersState) => {
        return this.usersService
          .getUsers()
          .pipe(
            map(
              (users: User[]) => new usersActions.LoadUsersSuccessAction(users)
            )
          );
      },

      onError: (action: usersActions.LoadUsersAction, error) =>
        new usersActions.LoadUsersFailAction(error)
    }
  );

  constructor(
    private actions: Actions,
    private dataPersistence: DataPersistence<UsersState>,
    private usersService: UsersService
  ) {}
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Install ngrx's entity library

```text
npm install @ngrx/entity
```

* Add interface information 

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/users.reducer.ts" %}
```typescript
///----- ABBREVIATED CODE -----///

import { Action } from '@ngrx/store';
import { UsersActions, UsersActionTypes } from './users.actions';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import { User } from '@demo-app/data-models';

/**
 * Interface for the 'Users' data used in
 *  - UsersState, and
 *  - usersReducer
 */
export interface UsersData extends EntityState<User> {
  selectedUserId: number;
  loading: boolean;
}

/**
 * Interface to the part of the Store containing UsersState
 * and other information related to UsersData.
 */
export interface UsersState {
  readonly users: UsersData;
}

export const adapter: EntityAdapter<User> = createEntityAdapter<User>();

export const initialState: UsersData = adapter.getInitialState({
  selectedUserId: null,
  loading: false
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add default state
* Update the default reducer logic

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/+state/users.reducer.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { UsersActions, UsersActionTypes } from './users.actions';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import { User } from '@demo-app/data-models';

/**
 * Interface for the 'Users' data used in
 *  - UsersState, and
 *  - usersReducer
 */
export interface UsersData extends EntityState<User> {
  selectedUserId: number;
  loading: boolean;
}

/**
 * Interface to the part of the Store containing UsersState
 * and other information related to UsersData.
 */
export interface UsersState {
  readonly users: UsersData;
}

export const adapter: EntityAdapter<User> = createEntityAdapter<User>();

export const initialState: UsersData = adapter.getInitialState({
  selectedUserId: null,
  loading: false
});

export function usersReducer(
  state = initialState,
  action: UsersActions
): UsersData {
  switch (action.type) {
    case UsersActionTypes.LoadUsers: {
      return { ...state, loading: true };
    }

    case UsersActionTypes.LoadUsersSuccess: {
      return adapter.addAll(action.payload, { ...state, loading: false });
    }

    default: {
      return state;
    }
  }
}

export const getSelectedUserId = (state: UsersData) => state.selectedUserId;

export const {
  // select the array of user ids
  selectIds: selectUserIds,

  // select the dictionary of user entities
  selectEntities: selectUserEntities,

  // select the array of users
  selectAll: selectAllUsers,

  // select the total user count
  selectTotal: selectUserTotal
} = adapter.getSelectors();

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add default selectors to use entity adapter

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/index.ts" %}
```typescript
import { createSelector, createFeatureSelector, ActionReducerMap } from '@ngrx/store';
import * as fromUsers from './users.reducer';
import { Users } from './users.interfaces';

export const selectUserState = createFeatureSelector<Users>('users');

export const selectUserIds = createSelector(selectUserState, fromUsers.selectUserIds);
export const selectUserEntities = createSelector(selectUserState, fromUsers.selectUserEntities);
export const selectAllUsers = createSelector(selectUserState, fromUsers.selectAllUsers);
export const selectUserTotal = createSelector(selectUserState, fromUsers.selectUserTotal);
export const selectCurrentUserId = createSelector(selectUserState, fromUsers.getSelectedUserId);

export const selectCurrentUser = createSelector(
  selectUserEntities,
  selectCurrentUserId,
  (userEntities, userId) => userEntities[userId]
);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Export via index.ts file the publicly available actions and selectors from the +state folder

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/index.ts" %}
```typescript
export { UsersModule } from './src/users.module';
export { UsersState } from './src/+state/users.reducer';
export * from './src/+state/index';
export * from './src/+state/users.actions';

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the logic to select and load all users via NgRx to the UserListComponent

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/containers/user-list/user-list.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { User } from '@demo-app/data-models';
import { UsersState, selectAllUsers, LoadUsersAction } from '@demo-app/admin-portal/users';

@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.scss']
})
export class UserListComponent implements OnInit {
  users$: Store<User[]>;

  constructor(private store: Store<UsersState>) {}

  ngOnInit() {
    this.store.  dispatch(new LoadUsersAction());
    this.users$ = this.store.select(selectAllUsers);
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Check the whole things works and dump the users onto the page

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/containers/user-list.component.html" %}
```markup
{{users$ | async | json}}
```
{% endcode-tabs-item %}
{% endcode-tabs %}



