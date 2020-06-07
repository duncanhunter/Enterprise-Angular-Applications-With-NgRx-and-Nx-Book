# 3 - Generating components and Nx lib

## 1. Generate our first Nx lib

* Run the below command to see all the lib options

```bash

```bash
>nx g lib --help
nx generate @nrwl/angular:library [name] [options,...]
Options:
  --name                  Library name
  --directory             A directory where the lib is placed
  --publishable           Generate a buildable library.
  --prefix                The prefix to apply to generated selectors.
  --skipFormat            Skip formatting files
  --simpleModuleName      Keep the module name simple (when using --directory)
  --skipPackageJson       Do not add dependencies to package.json.
  --skipTsConfig          Do not update tsconfig.json for development experience.
  --style                 The file extension to be used for style files. (default: css)
  --routing               Add router configuration. See lazy for more information.
  --lazy                  Add RouterModule.forChild when set to true, and a simple array of routes when set to false.
  --parentModule          Update the router configuration of the parent module using loadChildren or children, depending on what `lazy` is set to.
  --tags                  Add tags to the library (used for linting)
  --unitTestRunner        Test runner to use for unit tests (default: jest)
  --dryRun                Runs through and reports activity without writing to disk.
  --help                  Show available options for project target.
```

* Add a new lib called auth. We will not lazy load this lib as auth will always be used by our app.  Choose SASS as your styelsheet language.

```text
nx generate @nrwl/angular:lib auth --routing
```

The output should look like this:

```txt
? Which stylesheet format would you like to use? SASS(.scss)  [ http://sass-lang.com   ]
CREATE libs/auth/README.md (132 bytes)
CREATE libs/auth/tsconfig.lib.json (408 bytes)
CREATE libs/auth/tslint.json (247 bytes)
CREATE libs/auth/src/index.ts (35 bytes)
CREATE libs/auth/src/lib/auth.module.ts (269 bytes)
CREATE libs/auth/src/lib/auth.module.spec.ts (339 bytes)
CREATE libs/auth/tsconfig.json (123 bytes)
CREATE libs/auth/jest.config.js (347 bytes)
CREATE libs/auth/tsconfig.spec.json (233 bytes)
CREATE libs/auth/src/test-setup.ts (30 bytes)
UPDATE workspace.json (10310 bytes)
UPDATE nx.json (788 bytes)
UPDATE tsconfig.json (565 bytes)
```

* Inspect the index.ts file which is for exporting public api surface for the lib.
* Inspect angular.json libs array allow us to use --project flag with libs to get cli scaffolding and code generation.

{% hint style="info" %}
Currently issue with setting scss as default for lib. When ever you make a lib component please change style extention .scss
{% endhint %}

## 2. Add container and presentational components

Lets look at what this pattern is and what are the benefits in slides  
[https://docs.google.com/presentation/d/1xf8aPIvQjgjUVGH\_1sRkikvh5H73x2xvX7PnN4AjYt4/edit\#slide=id.g3bc936a676\_1\_4](https://docs.google.com/presentation/d/1xf8aPIvQjgjUVGH_1sRkikvh5H73x2xvX7PnN4AjYt4/edit#slide=id.g3bc936a676_1_4)

![Characteristics of Container and Presentational Components](.gitbook/assets/image%20%288%29.png)

* Add a new container component to the auth lib

```text
nx generate @nrwl/angular:component containers/login --project=auth
```

The output will look like this:

```txt
>nx generate @nrwl/angular:component containers/login --project=auth
CREATE libs/auth/src/lib/containers/login/login.component.html (20 bytes)
CREATE libs/auth/src/lib/containers/login/login.component.spec.ts (621 bytes)
CREATE libs/auth/src/lib/containers/login/login.component.ts (276 bytes)
CREATE libs/auth/src/lib/containers/login/login.component.css (0 bytes)
UPDATE libs/auth/src/lib/auth.module.ts (372 bytes)
```

* Add a new presentational component to the auth lib

```text
ng g c components/login-form --project=auth
```

Note: You may decide not to make this a presentational component but it makes it easier to test and refactor.

{% hint style="info" %}
As this is a child route it is assumed that the consuming module will already prefix the route with "localhost:4200/auth" and when we say the path is "login" we mean relative to this so it will end up being "localhost:4200/auth/login".
{% endhint %}

## 3. Add a default route to the auth module

{% code title="libs/auth/src/auth.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { LoginFormComponent } from './components/login-form/login-form.component';

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
@NgModule({
  imports: [CommonModule, RouterModule],
  declarations: [LoginComponent, LoginFormComponent]
})
export class AuthModule {}

```
{% endcode %}

## 4. Update the consuming Customer Portal App module

* Delete everything but the router-outlet on the apps app.component.html file.

{% code title="apps/customer-portal/src/app/app.components.html" %}
```typescript
<router-outlet></router-outlet>
```
{% endcode %}

* Add the Auth module to the main App Module

{% code title="apps/customer-portal/src/app/app.module.ts" %}
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';  //added

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    RouterModule.forRoot([{path: 'auth', children: authRoutes}], { initialNavigation: 'enabled' }),
    AuthModule .     // added
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

```
{% endcode %}

* Run the app `ng s` and navigate to [http://localhost:4200/auth/login](http://localhost:4200/auth/login)

![App running in the Browser](.gitbook/assets/image%20%2819%29.png)

## 4. Add presentational component to container component

* Add the presentational component to the container component.

{% code title="libs/auth/src/lib/containers/login/login.component.html" %}
```markup
<app-login-form (submit)="login($event)"></app-login-form>
```
{% endcode %}

* Add login function to container component class

{% code title="libs/auth/src/lib/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss']
})
export class LoginComponent implements OnInit {
  constructor() {}

  ngOnInit() {}

  login(authenticate:any) {
    console.log(authenticate);
  }
}
```
{% endcode %}

## 5. Add new folder for shared interfaces

* Make a folder called 'data-models' manually in the libs folder.

![data-models folder](.gitbook/assets/image%20%2811%29.png)

* Add a 'authenticate.d.ts' file to the folder and export the added data models from the index.ts file

{% code title="libs/data-models/src/authenticate.d.ts" %}
```typescript
export interface Authenticate {
    username: string;
    password: string;
}
```
{% endcode %}

* Export the interface

{% code title="libs/data-models/index.ts" %}
```typescript
export { Authenticate } from './authenticate';
```
{% endcode %}

* Add basic login form to presentational component

{% code title="libs/auth/src/lib/components/login-form/login-form.component.html" %}
```markup
<input #username placeholder="username" type="text">
<input #password placeholder="password" type="text">
<button (click)="login({username: username.value, password: password.value})">Login</button>
```
{% endcode %}

* Add an angular @Output to emit the event of a form submission

{% code title="libs/auth/src/lib/components/login-form/login-form.component.ts" %}
```typescript
import { Component, EventEmitter, Output } from '@angular/core';
import { Authenticate } from '@demo-app/data-models';
@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.scss']
})
export class LoginFormComponent {
  @Output() submit = new EventEmitter<Authenticate>();

  login(authenticate: Authenticate) {
    this.submit.emit(authenticate);
  }
}

```
{% endcode %}

## 6. Change the ChangeDetectionStrategy to OnPush

* Now that we are using the presentation and container component pattern and we know that we only need to check the child components for changes if a DOM event or a @Input or @Output passes new primitives or reference values. In this way we can tell Angular not check the whole component tree which can cause performance issues in larger applications.

{% code title="libs/auth/src/lib/containers/login/login.component.ts" %}
```typescript
import { Component, OnInit, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class LoginComponent implements OnInit {
  constructor() {}

  ngOnInit() {}

  login(authenticate: any) {
    console.log(authenticate);
  }
}

```
{% endcode %}

