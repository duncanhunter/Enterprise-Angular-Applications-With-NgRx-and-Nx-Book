---
description: In this section we will introduce and use feature modules
---

# 1c -  Brief Introduction To Angular

## 1. Refactor to use lazy loaded User module {#11-extra-add-extra-project-to-cli-app}

```text
ng g module user --routing
```

## 2.  Move all User related folders into the user module folder

![User feature folder](../.gitbook/assets/image.png)

## 3.  Remove all references to Users in App module

{% code-tabs %}
{% code-tabs-item title="src/app/app.module.ts" %}
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HomeComponent } from './home/home.component';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Change routes to lazy loaded routes

{% code-tabs %}
{% code-tabs-item title="src/app/app-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', loadChildren: 'src/app/user/user.module#UserModule' },   ///changed
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Add UserList component as feature module route

{% code-tabs %}
{% code-tabs-item title="src/app/user/user-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { UserListComponent } from './user-list/user-list.component'; // changed

const routes: Routes = [
  { path: '', component: UserListComponent },    //changed
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class UserRoutingModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 6. Declare UserList component in new User module

{% code-tabs %}
{% code-tabs-item title="src/app/user/user.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserListComponent } from './user-list/user-list.component';   //added
import { UserRoutingModule } from './user-routing.module';

@NgModule({
  imports: [
    CommonModule,
    UserRoutingModule
  ],
  declarations: [
    UserListComponent            //added
  ]
})
export class UserModule { }

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 7. Add button on App component to navigate to UserList or Home component

{% code-tabs %}
{% code-tabs-item title="src/app/app.component.html" %}
```markup
<h1>The App</h1>
​
<div>
  <button [routerLink]="['/']" routerLinkActive="router-link-active" >home</button>
  <button [routerLink]="['/users']" routerLinkActive="router-link-active" >users</button>
</div>

<!-- <app-home></app-home> -->
<router-outlet></router-outlet>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 8. Run the application and inspect the lazy loaded JavaScript

![Bundled feature module](../.gitbook/assets/image%20%288%29.png)

## Extras

### 1. Add extra project to CLI App

As of Angular CLI version 6+ you can now have multiple projects in an Angular app. It is early days and no support for sharing modules between apps and best practices or schematics made. That is what Nx can help with. You can read more on the Angular CLI limited docs ​[https://github.com/angular/angular-cli/wiki/stories-multiple-projects](https://github.com/angular/angular-cli/wiki/stories-multiple-projects)

In the terminal run

```text
ng generate application my-other-app
```

![Multi application support in default Angular CLI App](../.gitbook/assets/image%20%286%29.png)



### 2. Add npm package library to your app

  
Angular CLI v6 comes with library support via [ng-packagr](https://github.com/dherges/ng-packagr) plugged into the build system we use in Angular CLI, together with schematics for generating a library.

[https://github.com/angular/angular-cli/wiki/stories-create-library](https://github.com/angular/angular-cli/wiki/stories-create-library)

In the terminal run

```text
ng generate library my-lib
```

![Added npm library](../.gitbook/assets/image%20%2811%29.png)

