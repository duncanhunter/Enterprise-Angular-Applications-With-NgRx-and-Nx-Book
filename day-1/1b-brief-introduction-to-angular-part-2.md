---
description: In this section we will introduce Angular services and observables
---

# 1b -  Brief Introduction To Angular

## 1. Add a user service 

```text
ng g service services/user/user
```

## 2. Add HttpClientModule to AppModule

{% code-tabs %}
{% code-tabs-item title="src/app/app.module.ts" %}
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HomeComponent } from './home/home.component';
import { HttpClientModule } from '@angular/common/http'; // Added

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule     // Added
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Make a User service

Add some fake data in a user.json file in the assets folder of your new app. This is in place of a URL to hit a real API, for no.

{% code-tabs %}
{% code-tabs-item title="src/assets/users.json" %}
```javascript
[{ "name": "duncan" }, { "name": "sarah" }, { "name": "peter" }]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Make the service and inject in the HttpClient

{% code-tabs %}
{% code-tabs-item title="src/app/services/user/user.service.ts" %}
```typescript
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";

@Injectable({
  providedIn: 'root'
})
export class UserService {
  apiUrl = './../../../assets/users.json';

  constructor(private httpClient: HttpClient) {}

  getUsers(): Observable<any> { {
    return this.httpClient.get(this.apiUrl);
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Add a UserList component and inject in the User service

Make a user-list component, 'c' is short hand for component.

```text
ng g c user-list
```

Add a new eagerly loaded route for the UserList component

{% code-tabs %}
{% code-tabs-item title="src/app/app-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { UserListComponent } from './user-list/user-list.component'; //added

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'user-list', component: UserListComponent }, //added
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Check the route works by navigating to [http://localhost:4200/user-list](http://localhost:4200/user-list)

{% hint style="info" %}
Here we do the simple async pipe  
We will stop and discuss Observables and their operators in slides here [https://docs.google.com/presentation/d/1xf8aPIvQjgjUVGH\_1sRkikvh5H73x2xvX7PnN4AjYt4/edit?usp=sharing](https://docs.google.com/presentation/d/1xf8aPIvQjgjUVGH_1sRkikvh5H73x2xvX7PnN4AjYt4/edit?usp=sharing)
{% endhint %}

Inject user service and call its getUsers method and bind in to a local variable

{% code-tabs %}
{% code-tabs-item title="src/app/user-list/user-list.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { UserService } from '../services/user/user.service';

@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.css']
})
export class UserListComponent implements OnInit {
  users$: any;
  constructor(private userService: UserService) {}

  ngOnInit() {
    this.users$ = this.userService.getUsers();
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Use the async pipe to and an ngFor template directive to subscribe and iterate over the users.

{% code-tabs %}
{% code-tabs-item title="src/app/user-list/user-list.component.html" %}
```markup
<p>
  user-list works!
</p>
<div *ngFor="let user of (users$ | async)"> {{user.name}}</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Use Angular ngFor to bind users

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { UserService } from '../services/user/user.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {
  users$: any;

  constructor(private userService: UserService) { }

  ngOnInit() {
    this.users$ = this.userService.getUsers();
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 6. Add strong typing

Add User type to a models folder

```text
ng g interface models/user
```

{% code-tabs %}
{% code-tabs-item title="models/user.ts" %}
```typescript
export interface User {
  name: string;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add User type to the service

{% code-tabs %}
{% code-tabs-item title="src/app/services/user/user.service.ts" %}
```typescript
export class UserService {
  apiUrl = './../../../assets/users.json';            //changed
​
  constructor(private httpClient: HttpClient) {}
​
  getUsers(): Observable<User[]> {                    //changed
    return this.httpClient.get<User[]>(this.apiUrl);  //changed
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add User type to the UserList component

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { UserService } from '../services/user/user.service';
import { Observable } from 'rxjs';     //changed
import { User } from '../models/user'; //changed

@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.css']
})
export class UserListComponent implements OnInit {
  users$: Observable<User[]>;                      //changed

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.users$ = this.userService.getUsers();
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 

## Extra reading

Naming observables - [https://medium.com/@benlesh/observables-are-just-functions-but-also-collections-how-do-i-name-them-918c5ce2f64](https://medium.com/@benlesh/observables-are-just-functions-but-also-collections-how-do-i-name-them-918c5ce2f64)

