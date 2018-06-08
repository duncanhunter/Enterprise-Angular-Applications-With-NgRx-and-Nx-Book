---
description: In this section we will make a little local server
---

# 4 - Add JSON server

## 1. npm install json-server and ts-node

* add json-server and ts-node

```text
npm i json-server ts-node --save-dev
```

* Add a folder called server to the root directory and paste in the below code

## 2. Make a server.ts file in a new folder called server

* Make a new folder called server

![Server folder on root of project](../.gitbook/assets/image%20%2813%29.png)

* Make file called server.ts in the new server folder and paste in the below code. You do not need to study or learn this min node express app it is just to have a mock backend.

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

## 3. Add a file called db.json 

* Make file called db.json for mock data in the new server folder

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

## 4. Add the below script to the scripts in the package.json

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

## 5.  Start the server and leave it running

* Start the server and leave it running whenever you use the apps. It is easier to run in a seperate terminal to VS Code so when you refresh the IDE you do not kill this server.

```bash
npm run server
```

## 6. Navigate to [http://localhost:3000/login](http://localhost:3000/login) 

* Check it is working.

![](../.gitbook/assets/image%20%284%29.png)



