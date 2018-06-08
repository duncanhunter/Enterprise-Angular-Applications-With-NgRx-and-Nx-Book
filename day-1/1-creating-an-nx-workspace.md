# 1 -  Creating an Nx Workspace

We will be building out the beginning of two applications a customer portal and an admin portal.

![](https://github.com/duncanhunter/Enterprise-Angular-Applications-With-NgRx-and-Nx-Book/tree/d63a57a9f1ea36a7623cdf0746dd90b1406edaa2/.gitbook/assets/workspaces-demoapp.png)_**figure: nx workspaces diagram**_

## 1 .Create a new Nx workspace

* Make a new folder for your work space

```text
mkdir demo-app
```

* Run the below command in a terminal to make a new nx workspace.

```text
ng new demo-app --collection=@nrwl/schematics
```

## 2. Examine the output of the following files and commit code to git source control

1. Run the following command to open the new nx workspace in VSCode.

```text
cd demo-app && code .
```

* Inspect the following files:
  * angular.json
  * nx.json
  * tsconfig paths
  * package.json

## 3. Create a new app

* By default a new NX workspace has no apps or libs yet. You can run the below command to see the extra options to make an app or a lib besides the normal angular CLI commands.

```text
ng g app --help
```

* Create a new app by running the below command in the terminal in a directory of your choice. 

```text
ng g app customer-portal --style scss --routing --prefix ndc
```

## 4. Add NgRx

* Add a default set up for ngrx to our new app. We can run the generate command for ngrx with the module and --onlyEmptyRoot option to only add the StoreModule.forRoot and EffectsModule.forRoot calls without generating any new files versus --root which will make a default reducer and effect.

```text
ng g ngrx app --module=apps/customer-portal/src/app/app.module.ts  --onlyEmptyRoot
```

* Examine the AppModule to see added NgRx

{% code-tabs %}
{% code-tabs-item title="apps/customer-portal/src/app/app.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { BrowserModule } from '@angular/platform-browser';
import { NxModule } from '@nrwl/nx';
import { RouterModule } from '@angular/router';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';
import { StoreRouterConnectingModule } from '@ngrx/router-store';
import { storeFreeze } from 'ngrx-store-freeze';

@NgModule({
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    RouterModule.forRoot([], { initialNavigation: 'enabled' }),
    // Note: Issue with storeFreeze to be fixed in NgRx v6https://github.com/nrwl/nx/issues/436
    //StoreModule.forRoot({},{ metaReducers : !environment.production ? [storeFreeze] : [] }),
    StoreModule.forRoot({}),    EffectsModule.forRoot([]),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
 Note: StoreFreeze does not currently work with out extra steps. For this workshop we will disable by replacing the lines below.

Issue with storeFreeze to be fixed in NgRx v6 https://github.com/nrwl/nx/issues/436

```text
//StoreModule.forRoot({},{ metaReducers : !environment.production ? [storeFreeze] : [] }),
StoreModule.forRoot({}),    EffectsModule.forRoot([]),
```
{% endhint %}

## 6. Examine angular app and module structure

1. app.module.ts
2. tsconfig paths
3. package.json
4. apps and libs empty

## 7. Run the app in the browser

* Run the following command to launch the app in the browser. -a is for the app to start and -o is to open in the default browser.

```text
ng s -a=customer-portal -o
```

* See the default state of the app in the redux dev tools![](https://github.com/duncanhunter/Enterprise-Angular-Applications-With-NgRx-and-Nx-Book/tree/d63a57a9f1ea36a7623cdf0746dd90b1406edaa2/.gitbook/assets/default-ngrx-state.png)

## 8. Commit your code

* It is best to commit often when using the code generation tools so you can easily undo if it is not correct.



