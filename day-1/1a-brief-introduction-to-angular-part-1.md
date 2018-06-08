---
description: >-
  In this section we will create our first app and discuss event and property
  binding.
---

# 1a -  Brief Introduction To Angular

{% hint style="info" %}
The finished code for the next three parts can be found here https://github.com/duncanhunter/workshop-enterprise-angular-applications-with-ngrx-and-nx-cli-only
{% endhint %}



## 1. Create a new Angular CLI Project

Make a new workshop folder and generate a new CLI app

```text
mkdir workshop
```

```text
cd workshop
```

```text
ng new angular-cli-app --routing
```

Change directory into the new app's directory and open it with VS Code

```text
cd angular-cli-app
```

```text
code .
```

## 2. Run the app

Run the below command 's' is short for serve.

```text
ng s
```

Look at output.

![Bundled javascript files added to index.html dynamically](../.gitbook/assets/image%20%281%29.png)

## 3.  Review the structure and key files

package.json  
index.html  
main.ts  
app.module.ts  
app.component.ts  
app-routing.module.ts

## 4. Add a Home page component

Add home page component

```text
ng generate component home
```

Add home component selector to the AppComponent and delete all default HTML except the router outlet.

{% code-tabs %}
{% code-tabs-item title="src/app/app.component.html" %}
```markup
<h1>The App</h1>

<app-home></app-home>
<router-outlet></router-outlet>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Add a route

{% code-tabs %}
{% code-tabs-item title="src/app/spp-routing.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { } 
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 6. Event and data binding  

Add event and data binding to Home component with a new title property

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.html" %}
```markup
{{title}}

<input type="text" #input [value]="title" (keyup)="updateTitle(input.value)">

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Add a function to the Home component.

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {
  title: string;
  constructor() {}

  ngOnInit() {}

  updateTitle(value) {
    // debugger;
    console.log(`updateTitle: ${value}`);
    this.title = value;
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}



