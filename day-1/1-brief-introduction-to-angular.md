---
description: >-
  Before we dive into NgRx and Nx let's review the core parts of Angular and
  what a default Angular CLI project looks like.
---

# 1 -  Brief Introduction To Angular

## 1. Create a new Angular CLI Project

Make a new workshop folder

```text
mkdir workshop
cd workshop
```

```text
ng new angular-cli-app --routing
```

## 2. Run the app

```text
ng s
```

Look at output

## 3.  Add review the structure

package.json  
index.html  
main.ts  
app.module.ts  
app.component.ts  
app-routing.module.ts

## 4. Add a home page component

Add home page component

```text
ng generate component home
```

Add home component selector to the AppComponent and delete all default HTML except the router outlet.

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.ts" %}
```markup
<h1>The App</h1>

<app-home></app-home>
<router-outlet></router-outlet>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Add a route 

{% code-tabs %}
{% code-tabs-item title="src/app/spp-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 6. Add a service

```text
ng g service services/user/user
```

Add fake users JSON to src/assets folder. This is in pace or a URL to hit a real API.

{% code-tabs %}
{% code-tabs-item title="src/assets/user.json" %}
```javascript
[{ "name": "duncan" }, { "name": "sarah" }, { "name": "peter" }]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

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

  getUsers() {
    return this.httpClient.get(this.apiUrl);
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 7. Inject service into Home Component

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

## 8. Use Angular ngFor to bind users

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.html" %}
```markup
<p>
  home works!
</p>
<div *ngFor="let user of (users$ | async)"> {{user.name}}</div>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 9. Add strong typing

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

Add User type to component

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { UserService } from '../services/user/user.service';
import { User } from '../models/user';
import { Observable } from 'rxjs';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {
  users$: Observable<User[]>;

  constructor(private userService: UserService) { }

  ngOnInit() {
    this.users$ = this.userService.getUsers();
  }

}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add User type to the service

{% code-tabs %}
{% code-tabs-item title="src/app/services/user/user.service.ts" %}
```typescript
export class UserService {
  apiUrl = './../../../assets/users.json';

  constructor(private httpClient: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.httpClient.get<User[]>(this.apiUrl);
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 10. Extra: Add extra project to CLI App

As of Angular CLI version 6+ you can now have multiple projects in an Angular app. It is early days and no support for sharing modules between apps and best practices or schematics made. That is what Nx can help with. You can read more on the Angular CLI limited docs ​https://github.com/angular/angular-cli/wiki/stories-multiple-projects

In the terminal run

```text
ng generate application my-other-app
```

![Multi application support in default Angular CLI App](../.gitbook/assets/image%20%282%29.png)



