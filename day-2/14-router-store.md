# 14 - Router Store

## 1. Add a new presentational components for users

```text
ng g c components/users-table -a=admin-portal/users
```

* Add a toolbar module

```text
ng g c components/users-table-toolbar -a=admin-portal/users
```

* Add a toolbar with select field

{% hint style="danger" %}
Do not forget to add the MaterialModule to this module to use the toolbar
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/components/users-table-toolbar/users-table-toolbar.component.html" %}
```markup
<mat-toolbar>
  <mat-form-field >
    <mat-select [value]="selectedCountry" (selectionChange)="onFilter($event.value)" >
      <mat-option>None</mat-option>
      <mat-option value="australia" >Australia</mat-option>
      <mat-option value="england">England</mat-option>
      <mat-option value="usa">USA</mat-option>
    </mat-select>
  </mat-form-field>
</mat-toolbar>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/components/users-table-toolbar/users-table-toolbar.component.ts" %}
```typescript
import { Component, OnInit, Output, EventEmitter } from '@angular/core';


@Component({
  selector: 'app-users-table-toolbar',
  templateUrl: './users-table-toolbar.component.html',
  styleUrls: ['./users-table-toolbar.component.scss']
})
export class UsersTableToolbarComponent {
  @Output() filter = new EventEmitter<string>();
  selectedCountry = 'none';

  onFilter(country:string){
    this.filter.emit(country);
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add a users list table component

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/components/users-table/users-table.component.ts" %}
```typescript
import { Component, OnInit, Input, OnChanges, SimpleChanges } from '@angular/core';
import { User } from '@demo-app/data-models';
import { MatTableDataSource } from '@angular/material';


@Component({
  selector: 'app-users-table',
  templateUrl: './users-table.component.html',
  styleUrls: ['./users-table.component.scss']
})
export class UsersTableComponent implements OnChanges {
  @Input() users: User[];

  displayedColumns = ['username', 'country'];
  dataSource: MatTableDataSource<User>;

  ngOnChanges(changes: SimpleChanges) {
    this.dataSource = new MatTableDataSource(this.users);
  }

  setTableDatasource(dataSource): void {
    this.dataSource = new MatTableDataSource(this.users);
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the html

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/components/users-table/users-table.component.html" %}
```markup
<div class="mat-elevation-z8">
  <mat-table #table [dataSource]="dataSource">

    <ng-container matColumnDef="username">
      <mat-header-cell *matHeaderCellDef> Username </mat-header-cell>
      <mat-cell *matCellDef="let element"> {{element.username}} </mat-cell>
    </ng-container>

    <ng-container matColumnDef="country">
      <mat-header-cell *matHeaderCellDef> Country </mat-header-cell>
      <mat-cell *matCellDef="let element"> {{element.country}} </mat-cell>
    </ng-container>

    <mat-header-row *matHeaderRowDef="displayedColumns"></mat-header-row>
    <mat-row *matRowDef="let row; columns: displayedColumns;"></mat-row>
  </mat-table>
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 2. Add filter actions to update state

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/users.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { User } from '@demo-app/data-models';

export enum UsersActionTypes {
  LoadUsers = '[Users] Load',
  LoadUsersSuccess = '[Users] Load Sucess',
  LoadUsersFail = '[Users] Load Fail',
  SetUsersFilter = '[Users] Set Filter'
}

export class LoadUsersAction implements Action {
  readonly type = UsersActionTypes.LoadUsers;
}

export class LoadUsersSuccessAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(public payload: User[]) {}
}

export class LoadUsersFailAction implements Action {
  readonly type = UsersActionTypes.LoadUsersSuccess;
  constructor(public payload: any) {}
}

export class SetUsersFiltersAction {
  readonly type = UsersActionTypes.SetUsersFilter;
  constructor(public payload: string) {}
}

export type UsersActions =
  | LoadUsersAction
  | LoadUsersSuccessAction
  | LoadUsersFailAction
  | SetUsersFiltersAction;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Update the users state to have a selectedCountry 

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/users.reducer.ts" %}
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
  selectedCountry: string
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
  loading: false,
  selectedCountry: 'none'
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

* Remove load dispatch action from component

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/containers/user-list/user-list.component.ts" %}
```typescript
ngOnInit() {
  // this.store.dispatch(new usersActions.LoadUsersAction());
  this.users$ = this.store.select(selectAllUsers);
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Add router method to user-list component

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/containers/user-list/user-list.component.html" %}
```text
<app-users-table-toolbar (filter)="updateUrlFilters($event)"></app-users-table-toolbar>
<app-users-table [users]="users$ | async"></app-users-table>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/containers/user-list/user-list.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { User } from '@demo-app/data-models';
import { UsersState } from '../../+state/users.reducer';
import * as usersActions from '../../+state/users.actions';
import { selectAllUsers } from '../../+state';
import { NavigationExtras, Router } from '@angular/router';

@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.scss']
})
export class UserListComponent implements OnInit {
  users$: Store<User[]>;

  constructor(private router: Router, private store: Store<UsersState>) {}

  ngOnInit() {
    // this.store.  dispatch(new LoadUsersAction());
    this.users$ = this.store.select(selectAllUsers);
  }

  updateUrlFilters(country: string): void {
    const navigationExtras: NavigationExtras = {
      replaceUrl: true,
      queryParams: {country}
    };
    this.router.navigate([`/users`], navigationExtras);
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add an effect to listen for the route change

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/+state/users.effects.ts" %}
```typescript
///------ ABBREVIATED CODE -----///

  @Effect()
  loadUsersFromRoute = this.dataPersistence.navigation(UserListComponent, {
    run: (a: ActivatedRouteSnapshot, state: UsersState) => {
      return this.usersService
        .getUsers(a.queryParams['country'])
        .pipe(
          map((users: User[]) => new usersActions.LoadUsersSuccessAction(users))
        );
    },
    onError: (a: ActivatedRouteSnapshot, e: any) => {
      return new usersActions.LoadUsersFailAction(e);
    }
  });
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Update the user service

{% code-tabs %}
{% code-tabs-item title="libs/admin-portal/users/src/services/users.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()

export class UsersService {
  constructor(private httpClient: HttpClient) {}

  getUsers(country: string = null) {
    const url = country !== null
      ? `http://localhost:3000/users?country=${country}`
      : `http://localhost:3000/users`;

    return this.httpClient.get(url);
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

