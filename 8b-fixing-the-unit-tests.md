---
description: >-
  In this section we fix the unit tests for the auth module.
---

# Fixing the unit tests

At this point, if you run all the tests for the auth lib, it has some failing tests.

```bash
nx test auth --watch
...
Tests:       3 failed, 1 passed, 4 total
```

The first failure is:

```bash
FAIL  libs/auth/src/lib/services/auth/auth.service.spec.ts
  ● AuthService › should be created
    NullInjectorError: StaticInjectorError(DynamicTestModule)[HttpClient]:
      StaticInjectorError(Platform: core)[HttpClient]:
        NullInjectorError: No provider for HttpClient!
```

This is an easy fix as the test just needs its dependencies updated.

```js
import { HttpClientTestingModule } from '@angular/common/http/testing';
...
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule]});
      ...
```

That fixes two of the failures. The last one is also a missing dependency:

```bash
FAIL  libs/auth/src/lib/containers/login/login.component.spec.ts
  ● LoginComponent › should create
    Template parse errors:
    'app-login-form' is not a known element:
    1. If 'app-login-form' is an Angular component, then verify that it is part of this module.
    2. If 'app-login-form' is a Web Component then add 'CUSTOM_ELEMENTS_SCHEMA' to the '@NgModule.schemas' of this component to suppress this message. ("<br />
```

Run the tests again now shows two failures:

```bash
nx run auth:test --watch=true
...
  ● LoginComponent › should create
    Unexpected directive 'LoginFormComponent' imported by the module 'DynamicTestModule'. Please add a @NgModule annotation.
...
    Template parse errors:
    'mat-card-title' is not a known element:
    1. If 'mat-card-title' is an Angular component, then verify that it is part of this module.
```

The last one requires MaterialModule to be imported by the spec.

After that we see another failure:

```bash
    Template parse errors:
    Can't bind to 'formGroup' since it isn't a known property of 'form'. ("
      <mat-card-title>Login</mat-card-title>
      <mat-card-content>
        <form [ERROR ->][formGroup]="loginForm" fxLayout="column" fxLayoutAlign="center none">
```

This of course requires the forms stuff added to the imports array:

```js
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
```

You might also get a failure like this:

```bash
  ● LoginFormComponent › should create
    Found the synthetic property @transitionMessages. Please include either "BrowserAnimationsModule" or "NoopAnimationsModule" in your application.
```

Since @transitionMessages is not found in the project, it must be part of material which we just imported above. Stopping and starting the tests and closing and opening VSCode fixes this. Or, fixes one of them. Now the only test failing is the @transitionMessages one.

We have BrowserAnimationsModule imported in the app.module.ts file. [This issue](https://github.com/angular/angular/issues/18751) is still open on the Angular GitHub.

The solution from [this blog](https://onlyangular5.blogspot.com/2018/02/complete-angular-5-tutorial-for.html): _Use (submit) instead of (ngSubmit)._

We don't have a submit in the login form component. We do have this however:

```js
@Output() submit = new EventEmitter<Authenticate>();
```

There is a TypeScript warning on this: _In the class "LoginFormComponent", the output property "submit" should not be named or renamed as a native event (no-output-native)tslint(1)_

That was noticed before. Changed submit to submitLogin and the warning is gone. Also imported these in the login.component.spec.ts file, same as the form, and added them to the imports array.

```js
import { MaterialModule } from '@clades/material';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
```

Now the login component has this failure:

```bash
NullInjectorError: No provider for HttpClient!
```

This requires the testing module imported and added to the array:

```js
import { HttpClientTestingModule } from '@angular/common/http/testing';
```

Now both login and login-form component specs are failing with the _Found the synthetic property @transitionMessages._.

Import both MaterialModule and BrowserAnimationsModule in both failing specs and the tests pass!
