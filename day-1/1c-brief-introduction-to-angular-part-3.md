---
description: In this section we will introduce and use feature modules
---

# 1c -  Brief Introduction To Angular Part 3

## 1. Refactor to use lazy loaded User module {#11-extra-add-extra-project-to-cli-app}

```text
ng g module user --routing
```

## 2.  Move all User related folders into the user module folder

![User feature folder](../.gitbook/assets/image.png)

## 3. Change routes to lazy loaded routes

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

## Extra: Add extra project to CLI App

As of Angular CLI version 6+ you can now have multiple projects in an Angular app. It is early days and no support for sharing modules between apps and best practices or schematics made. That is what Nx can help with. You can read more on the Angular CLI limited docs ​[https://github.com/angular/angular-cli/wiki/stories-multiple-projects](https://github.com/angular/angular-cli/wiki/stories-multiple-projects)

In the terminal run

```text
ng generate application my-other-app
```

![Multi application support in default Angular CLI App](../.gitbook/assets/image%20%285%29.png)

