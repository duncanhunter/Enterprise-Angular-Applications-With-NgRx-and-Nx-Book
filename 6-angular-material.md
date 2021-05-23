---
description: >-
  In this section we will explore Angular Material and installing third party
  dependencies
---

# 6 - Angular Material

![Angular Material Website](.gitbook/assets/material-site.png)

[https://material.angular.io/](https://material.angular.io/)

## 1. Install angular material, angular animations and angular flex layout

{% hint style="info" %}
Note: As of Angular v6 you no longer need to manually add Angular material you can use the new "Add" CLI command. However this is not how you do it with Nx.

Always use the same Major version of Material as your Angular CLI and packages.
{% endhint %}

```text
npm install @angular/material @angular/cdk @angular/flex-layout @angular/animations --save
```

* Add animations module to the main app module.

{% code title="src/app/app.module.ts" %}

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { authRoutes, AuthModule } from '@demo-app/auth';
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';   // Added

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,        // Added
    NxModule.forRoot(),
    RouterModule.forRoot([{path: 'auth', children: authRoutes}], { initialNavigation: 'enabled' }),
    AuthModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

```

{% endcode %}

## 2. Add a new Nx lib to hold all the common material components we will use in our apps

```bash
nx generate @nrwl/angular:lib material
```

* Add all the common material components and re-export them

{% code title="libs/material/src/lib/material.module.ts" %}

```typescript
import { NgModule } from '@angular/core';
import { FlexLayoutModule } from '@angular/flex-layout';
import { MatInputModule } from '@angular/material/input';
import { MatCardModule } from '@angular/material/card';
import { MatButtonModule } from '@angular/material/button';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';
import { MatIconModule } from '@angular/material/icon';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';
import { MatMenuModule } from '@angular/material/menu';
import { MatTableModule } from '@angular/material/table';
import { MatSelectModule } from '@angular/material/select';

@NgModule({
  imports: [
    FlexLayoutModule,
    MatInputModule,
    MatCardModule,
    MatButtonModule,
    MatSidenavModule,
    MatListModule,
    MatIconModule,
    MatToolbarModule,
    MatProgressSpinnerModule,
    MatMenuModule,
    MatTableModule,
    MatSelectModule
  ],
  exports: [
    FlexLayoutModule,
    MatInputModule,
    MatCardModule,
    MatButtonModule,
    MatSidenavModule,
    MatListModule,
    MatIconModule,
    MatToolbarModule,
    MatProgressSpinnerModule,
    MatMenuModule,
    MatTableModule,
    MatSelectModule
  ]
})
export class MaterialModule {}
```

{% endcode %}

* Add material module to auth module

{% code title="libs/auth/src/auth.module.ts" %}

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Route } from '@angular/router';
import { LoginComponent } from './containers/login/login.component';
import { LoginFormComponent } from './components/login-form/login-form.component';
import { HttpClientModule } from '@angular/common/http';
import { MaterialModule } from '@demo-app/material';   // Added

export const authRoutes: Route[] = [
  { path: 'login', component: LoginComponent }
];
@NgModule({
  imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule], // Added
  declarations: [LoginComponent, LoginFormComponent]
})
export class AuthModule {}

```

{% endcode %}

## 3. Add material default styles

{% code title="apps/customer-portal/src/styles.scss" %}

```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';
```

{% endcode %}

* Update the login-form to use material components

{% code title="libs/auth/src/lib/components/login-form/login-form.component.html" %}

```markup
<mat-card>
    <mat-card-title>Login</mat-card-title>
    <mat-card-content>
        <form fxLayout="column" fxLayoutAlign="center none">
            <mat-form-field>
                <input matInput placeholder="username" type="text" #username>
            </mat-form-field>
            <mat-form-field>
                <input matInput placeholder="password" type="text" #password>
            </mat-form-field>
        </form>
        <button mat-raised-button (click)="login({username:username.value, password:password.value})">login</button>
    </mat-card-content>
</mat-card>
```

{% endcode %}

## 4. Go and explore flex layout docs

* Go to this website and try out the flex examples.

[https://tburleson-layouts-demos.firebaseapp.com/\#/docs](https://tburleson-layouts-demos.firebaseapp.com/#/docs)

![flex-layout library examples](.gitbook/assets/image%20%2821%29.png)
