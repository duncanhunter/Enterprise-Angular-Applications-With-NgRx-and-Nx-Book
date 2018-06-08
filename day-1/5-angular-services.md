---
description: In this section we discussing Angular services and dependency injection
---

# 5 - Angular Services

## 1. Generate a new angular service

* Run the following command to make a new service in the auth lib

```text
ng g service services/auth/auth --project=auth
```

## 2. Add login method and http post for the login

* Add HttpClientModule to Auth Module

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/lib/auth.module.ts" %}
```typescript
/// abbreviated code
@NgModule({
  imports: [CommonModule, RouterModule, HttpClientModule],
  declarations: [LoginComponent, LoginFormComponent]
})
export class AuthModule {}

/// abbreviated code
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add login call to a local server we will add shortly

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/lib/services/auth/auth.service.ts" %}
```typescript
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';

@Injectable()
export class AuthService {
  constructor(private httpClient: HttpClient) {}

  login(authenticate:Authenticate) {
    return this.httpClient.post('http://localhost:3000/login', authenticate);
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Export the auth service from the auth lib and add a static for root method.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/auth.module.ts" %}
```typescript
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { HttpClientModule } from '@angular/common/http';
import { AuthService } from './services/auth/auth.service';
import { LoginFormComponent } from './components/login-form/login-form.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
const COMPONENTS = [LoginComponent, LoginFormComponent];

@NgModule({
  imports: [CommonModule, RouterModule, HttpClientModule],
  declarations: [COMPONENTS],
  exports: [COMPONENTS],
  providers: [AuthService]
})
export class AuthModule {}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Update login component to call the service

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { AuthService } from './../../services/auth/auth.service';
import { Authenticate } from '@demo-app/data-models';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor(private authService: AuthService) {}

  ngOnInit() {}

  login(authenticate: Authenticate) {
    this.authService.login(authenticate).subscribe()
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
Note the .subscribe\(\) is needed to make sure the observer is registered with the observable returned from our AuthService.
{% endhint %}

## 4. Attempt to login with default users

* To login as an admin use the below credentials and this will return a fake admin token. Note this is in no means an attempt to make a production authentication service it is purely to give us mock data from a real angular HTTP request.

```text
username: admin
password: 123
```

* To login as a non-admin use the below credentials

```text
username: duncan
password: 123
```

