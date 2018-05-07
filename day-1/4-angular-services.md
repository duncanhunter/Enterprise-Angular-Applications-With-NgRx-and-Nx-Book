# 4 - Angular Services

## 1. Generate a new angular service

* Run the following command to make a new service in the auth lib

```text
ng g service services/auth --flat=false -a=auth
```

## 2. Add login method and http post for the login

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/services/auth/auth.service.ts" %}
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

{% hint style="info" %}
As of Angular v6 you can register your providers in your service which makes them tree shack-able leading to smaller bundles loaded into the browser. 

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/services/auth/auth.service.ts" %}
```typescript
@Injectable({
    provideIn: AuthModule
})
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endhint %}

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
{% code-tabs-item title="libs/auth/src/containers/login/login.component.html" %}
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

## 4. Add a json-server to be able to make http requests and mock a real server

* Add a folder called server to the root directory
* Add a file called db.json and server.ts to the new server folder
* add json-server and ts-node

```text
npm i json-server ts-node --save-dev
```

* Add the below script to the scripts in the package.json

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
scripts: {
   ...
    "server": "ts-node ./server/server.ts"
   ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the default mock data to the db.json file

{% code-tabs %}
{% code-tabs-item title="server/db.json" %}
```javascript
{
    "users": [
      {
        "id": 1,
        "username": "duncan",
        "country": "australia",
        "password": "123"
      },
      {
        "id": 2,
        "username": "sarah",
        "country": "england",
        "password": "123"
      },
      {
        "id": 3,
        "username": "admin",
        "country": "usa",
        "password": "123"
      },
      {
        "username": "test2",
        "password": "123",
        "id": 4
      }
    ],
    "profiles": [
      {
        "id": 1,
        "userId": 1,
        "job": "plumber"
      },
      {
        "id": 2,
        "userId": 2,
        "job": "developer"
      },
      {
        "id": 3,
        "userId": 3,
        "job": "manager"
      }
    ]
  }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add the mock server code below

{% code-tabs %}
{% code-tabs-item title="server/server.ts" %}
```typescript
const jsonServer = require('json-server');
const server = jsonServer.create();
const router = jsonServer.router('server/db.json');
const middlewares = jsonServer.defaults();
const db = require('./db.json');
const fs = require('fs');

server.use(middlewares);
server.use(jsonServer.bodyParser);

server.post('/login', (req, res, next) => { 
  const users = readUsers();

  const user = users.filter(
    u => u.username === req.body.username && u.password === req.body.password
  )[0];

  if (user) {
    res.send({ ...formatUser(user), token: checkIfAdmin(user) });
  } else {
    res.status(401).send('Incorrect username or password');
  }
});

server.post('/register', (req, res) => {
  const users = readUsers();
  const user = users.filter(u => u.username === req.body.username)[0];

  if (user === undefined || user === null) {
    res.send({
      ...formatUser(req.body),
      token: checkIfAdmin(req.body)
    });
    db.users.push(req.body);
  } else {
    res.status(500).send('User already exists');
  }
});

server.use('/users', (req, res, next) => {
  if (isAuthorized(req) || req.query.bypassAuth === 'true') {
    next();
  } else {
    res.sendStatus(401);
  }
});

server.use(router);
server.listen(3000, () => {
  console.log('JSON Server is running');
});

function formatUser(user) {
  delete user.password;
  user.role = user.username === 'admin'
    ? 'admin'
    : 'user';
  return user;
}

function checkIfAdmin(user, bypassToken = false) {
  return user.username === 'admin' || bypassToken === true
    ? 'admin-token'
    : 'user-token';
}

function isAuthorized(req) {
  return req.headers.authorization === 'admin-token' ? true : false;
}

function readUsers() {
  const dbRaw = fs.readFileSync('./server/db.json');  
  const users = JSON.parse(dbRaw).users
  return users;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Start the server and leave it running whenever you use the apps.

```bash
npm run server
```

* Navigate to [http://localhost:3000/login](http://localhost:3000/login) to check it is working

## 5. Attempt to login with default users

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

