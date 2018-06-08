# 1b -  Brief Introduction To Angular Part 2

## 1. Add a user service 

```text
ng g services/user/user.service.ts
```

```text

```

## 

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

## 

## 7. Inject service into User Component

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

{% code-tabs %}
{% code-tabs-item title="src/assets/user.json" %}
```javascript
[{ "name": "duncan" }, { "name": "sarah" }, { "name": "peter" }]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add fake users JSON to src/assets folder. This is in pace or a URL to hit a real API.

```text
ng g service services/user/user
```



## 8. Use Angular ngFor to bind users

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

## 9. Add strong typing

{% code-tabs %}
{% code-tabs-item title="src/app/services/user/user.service.ts" %}
```typescript
export class UserService {
  apiUrl = './../../../assets/users.json';
​
  constructor(private httpClient: HttpClient) {}
​
  getUsers(): Observable<User[]> {
    return this.httpClient.get<User[]>(this.apiUrl);
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add User type to the service

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

## Add User type to component

{% code-tabs %}
{% code-tabs-item title="models/user.ts" %}
```typescript
export interface User {
  name: string;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```text
ng g interface models/user
```



