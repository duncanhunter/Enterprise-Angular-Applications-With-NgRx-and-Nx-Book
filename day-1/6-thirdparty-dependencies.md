# 6 - Thirdparty Dependencies

![Angular Material Website](../.gitbook/assets/material-site.png)

[https://material.angular.io/](https://material.angular.io/)

## 1.Install angular material, angular animations and angular flex layout

{% hint style="info" %}
Note: As of Angular v6 you no longer need to manually add Angular material you can use the new "Add" CLI command.

Always use the same Major version of Material as your Angular CLI and packages.

```bash
ng add @angular/material
```
{% endhint %}

```text
npm install --save @angular/material@5.2.5 @angular/cdk@5.2.5 @angular/flex-layout @angular/animations
```

* Add animations module to the main app module.

```typescript
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';

@NgModule({
  ...
  imports: [BrowserAnimationsModule],
  ...
})
export class AppModule { }
```

## 2. Add a new nx lib to hold all the common material components we will use in our app

```bash
ng g lib material
```

* Add all the common material components and re-export them

{% code-tabs %}
{% code-tabs-item title="libs/material/src/material.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { FlexLayoutModule } from '@angular/flex-layout';

import {
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
} from '@angular/material';

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
{% endcode-tabs-item %}
{% endcode-tabs %}

* Add material module to auth module

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/auth.module.ts" %}
```typescript
import { MaterialModule } from '@demo-app/material';

@NgModule({
  ...
imports: [CommonModule, RouterModule, HttpClientModule, MaterialModule],
  ...
})
export class AuthModule { }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 3. Add material default styles

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/styles.scss" %}
```css
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Update the login-form to use material components

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/components/login-form/login-form.component.html" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Go and explore flex layout docs

[https://tburleson-layouts-demos.firebaseapp.com/\#/docs](https://tburleson-layouts-demos.firebaseapp.com/#/docs)

