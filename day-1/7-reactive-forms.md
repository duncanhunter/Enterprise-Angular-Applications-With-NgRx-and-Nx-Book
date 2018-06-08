# 6 - Reactive Forms

## 1.Add ngx-errors library to make it easier to display form errors

[https://github.com/UltimateAngular/ngx-errors](https://github.com/UltimateAngular/ngx-errors)

```text
npm i @ultimate/ngxerrors
```

1. Add reactive forms module to auth module imports

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/auth.module.ts" %}
```typescript
imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule, ReactiveFormsModule],
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 2. Add a reactive FormGroup to login form

Note: To save injecting the formBuilder and keeping this a presentational component with no injected dependancies we can just new up a simple FormGroup. You can read more about it here[ https://angular.io/api/forms/FormBuilder](https://angular.io/api/forms/FormBuilder).

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/components/login-form/login-form.component.ts" %}
```typescript
import { Component, OnInit, EventEmitter, Output } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.scss']
})
export class LoginFormComponent {
  @Output() submit = new EventEmitter<Authenticate>();

  loginForm = new FormGroup({
    username: new FormControl('', [Validators.required]),
    password: new FormControl('', [Validators.required])
  });

  login() {
    this.submit.emit({
      username: this.loginForm.value.username,
      password: this.loginForm.value.password
    } as Authenticate);
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add ngx-errors to the form

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/components/login-form/login-form.component.html" %}
```markup
<mat-card>
  <mat-card-title>Login</mat-card-title>
  <mat-card-content>
      <form [formGroup]="loginForm" fxLayout="column" fxLayoutAlign="center none">
          <mat-form-field>
              <input matInput placeholder="username" type="text" formControlName="username">
              <mat-error>
                <div ngxError="required" when="touched">
                  Username is required
                </div>
              </mat-error>
          </mat-form-field>
          <mat-form-field>
              <input matInput placeholder="password" type="text" formControlName="password">
              <mat-error>
                <div ngxError="required" when="touched">
                  Password is required
                </div>
              </mat-error>
          </mat-form-field>
      </form>
      <button mat-raised-button (click)="login()">login</button>
  </mat-card-content>
</mat-card>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Add a User interface

* Add a User interface to strongly type the return value of the Auth Service.

{% code-tabs %}
{% code-tabs-item title="libs/data-models/src/data-models.ts" %}
```typescript
export interface Authenticate {
  username: string;
  password: string;
}

export interface User {
  username: string;
  id: number;
  country: string;
  token: string
  role: string;
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="libs/data-models/index.ts" %}
```typescript
export { Authenticate, User } from './src/data-models';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/services/auth/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class AuthService {
  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate): Observable<User> {
    return this.httpClient.post<User>(
      'http://localhost:3000/login',
      authenticate
    );
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

