---
description: In this section we discuss angular service
---

# 5 - Angular Services

## 1. Generate a new angular service

* Run the following command to make a new service in the auth lib

```text
nx generate @nrwl/angular:service services/auth/auth
```

## 2. Add login method and http post for the login

* Add **HttpClientModule** to Auth Module

{% code title="libs/auth/src/lib/auth.module.ts" %}
```typescript
/// abbreviated code 

import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [CommonModule, RouterModule, HttpClientModule],
  declarations: [LoginComponent, LoginFormComponent]
})
export class AuthModule {}

/// abbreviated code
```
{% endcode %}

* Add login call to a local server

{% code title="libs/auth/src/lib/services/auth/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate): Observable<any> {
    return this.httpClient.post('http://localhost:3000/login', authenticate);
  }
}

```
{% endcode %}

## 3. Update Login component to call the service

{% code title="libs/auth/src/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit, ChangeDetectionStrategy } from '@angular/core';
import { AuthService } from './../../services/auth/auth.service';
import { Authenticate } from '@demo-app/data-models';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class LoginComponent implements OnInit {
  constructor(private authService: AuthService) {}

  ngOnInit() {}

  login(authenticate: Authenticate): void {
    this.authService.login(authenticate).subscribe();
  }
}

```
{% endcode %}

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

![Chrome Devtools](.gitbook/assets/image%20%2813%29.png)



