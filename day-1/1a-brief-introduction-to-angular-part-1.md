# 1a -  Brief Introduction To Angular Part 1

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

![Bundled javascript files added to index.html dynamically](../.gitbook/assets/image.png)

## 3.  Add review the structure

package.json  
index.html  
main.ts  
app.module.ts  
app.component.ts  
app-routing.module.ts

## 4. Add a home page component

Add home page component

```text
ng generate component home
```

Add home component selector to the AppComponent and delete all default HTML except the router outlet.

{% code-tabs %}
{% code-tabs-item title="src/app/home/home.component.ts" %}
```markup
<h1>The App</h1>

<app-home></app-home>
<router-outlet></router-outlet>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4. Add a route

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

## 5. Event and data binding  

Add event and data binding to App Component title

{% code-tabs %}
{% code-tabs-item title="src/app/app.component.html" %}
```markup
<h1>{{title}}</h1>

<input #nameInput type="text" [value]="title" (keyup)="updateTitle(nameInput.value)">

<app-home></app-home>
<router-outlet></router-outlet>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="src/app/app.component.ts" %}
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';

  updateTitle(value: string) {
    debugger;
    this.title = value;
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}



