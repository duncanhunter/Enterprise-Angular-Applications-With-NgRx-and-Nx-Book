---
description: In this section we examine Reactive forms.
---

# 7 - Reactive Forms

## 1. Add ReactiveFormsModule to Auth module

{% code title="libs/auth/src/lib/auth.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { LoginFormComponent } from './components/login-form/login-form.component';
import { HttpClientModule } from '@angular/common/http';
import { MaterialModule } from '@demo-app/material';
import { ReactiveFormsModule } from '@angular/forms';  // Added

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
@NgModule({
  imports: [
    CommonModule,
    RouterModule,
    HttpClientModule,
    MaterialModule,
    ReactiveFormsModule   // added
  ],
  declarations: [LoginComponent, LoginFormComponent]
})
export class AuthModule {}

```
{% endcode %}

## 2. Add a Reactive FormGroup to Login Form

Note: To save injecting the formBuilder and keeping this a presentational component with no injected dependancies we can just new up a simple FormGroup. You can read more about it here[ https://angular.io/api/forms/FormBuilder](https://angular.io/api/forms/FormBuilder).

{% code title="libs/auth/src/lib/components/login-form/login-form.component.ts" %}
```typescript
import { Component, EventEmitter, Output } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.scss']
})
export class LoginFormComponent {
  @Output() submitForm = new EventEmitter<Authenticate>();

  loginForm = new FormGroup({
    username: new FormControl('', [Validators.required]),
    password: new FormControl('', [Validators.required])
  });

  login() {
    this.submitForm.emit({
      username: this.loginForm.value.username,
      password: this.loginForm.value.password
    } as Authenticate);
  }
}
```
{% endcode %}

## 3. Add HTML markup to the form

{% code title="libs/auth/src/lib/components/login-form/login-form.component.html" %}
```markup
<mat-card>
  <mat-card-title>Login</mat-card-title>
  <mat-card-content>
    <form [formGroup]="loginForm" fxLayout="column" fxLayoutAlign="center none">
      <mat-form-field>
        <input matInput placeholder="username" type="text" formControlName="username">
        <mat-error>
          <span *ngIf="loginForm.get('username').hasError('required') && loginForm.touched">Required Field</span>
        </mat-error>
      </mat-form-field>
      <mat-form-field>
        <input matInput placeholder="password" type="text" formControlName="password">
        <mat-error>
          <span *ngIf="loginForm.get('password').hasError('required') && loginForm.touched">Required Field</span>
        </mat-error>
      </mat-form-field>
    </form>
    <button mat-raised-button (click)="login()">login</button>
  </mat-card-content>
</mat-card>

```
{% endcode %}

## 4. Add a User interface

* Add a User interface to the data-models folder to strongly type the return value of the Auth Service.

{% code title="libs/data-models/src/lib/user.d.ts" %}
```typescript
export interface User {
  username: string;
  id: number;
  country: string;
  token: string;
  role: string;
}
```
{% endcode %}

* Re-export the interface

{% code title="libs/data-models/src/lib/data-models.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
export { Authenticate } from './authenticate';
export { User } from './user';

@NgModule({
  imports: [CommonModule],
})
export class DataModelsModule {}

```
{% endcode %}

## 5. Add types to Auth service

{% code title="libs/auth/src/lib/services/auth/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
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
{% endcode %}

## 6. Check validation in the browser

![Form validation errors](.gitbook/assets/image.png)



