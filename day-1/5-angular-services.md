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
import { Injectable } from '@angular/core';
import { AuthModule } from '@demo-app/auth/src/lib/auth.module';
import { Authenticate } from '@demo-app/data-models';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: AuthModule
})
export class AuthService {
  constructor(private httpClient: HttpClient) {}
  login(authenticate: Authenticate) {
    return this.httpClient.post('http://localhost:3000/login', authenticate);
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Update Login component to call the service

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

Later in the workshop we will learn to use NgRx to get entities from a server, but for now this is normal angular code without NgRx
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

![Chrome Devtools](../.gitbook/assets/image%20%2812%29.png)

