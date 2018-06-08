# 8 - NgRx Introduction

## 1. Presentation

Presentation: NgRx

[https://docs.google.com/presentation/d/1MknPww1MdwzreFvDh086TzqaB5yyQga0PaoRR9bgCXs/edit?usp=sharing](https://docs.google.com/presentation/d/1MknPww1MdwzreFvDh086TzqaB5yyQga0PaoRR9bgCXs/edit?usp=sharing)



## 2. Add NgRx {#4-add-ngrx}

* Add a default set up for ngrx to our new app. We can run the generate command for ngrx with the module and --onlyEmptyRoot option to only add the StoreModule.forRoot and EffectsModule.forRoot calls without generating any new files versus --root which will make a default reducer and effect.

```text
ng g ngrx app --module=apps/customer-portal/src/app/app.module.ts  --onlyEmptyRoot
```

* Examine the AppModule to see added NgRx

Add fileTypeScript 

```text
import { NgModule } from '@angular/core';import { AppComponent } from './app.component';import { BrowserModule } from '@angular/platform-browser';import { NxModule } from '@nrwl/nx';import { RouterModule } from '@angular/router';import { StoreModule } from '@ngrx/store';import { EffectsModule } from '@ngrx/effects';import { StoreDevtoolsModule } from '@ngrx/store-devtools';import { environment } from '../environments/environment';import { StoreRouterConnectingModule } from '@ngrx/router-store';import { storeFreeze } from 'ngrx-store-freeze';​@NgModule({  imports: [    BrowserModule,    NxModule.forRoot(),    RouterModule.forRoot([], { initialNavigation: 'enabled' }),    // Note: Issue with storeFreeze to be fixed in NgRx v6https://github.com/nrwl/nx/issues/436    //StoreModule.forRoot({},{ metaReducers : !environment.production ? [storeFreeze] : [] }),    StoreModule.forRoot({}),    EffectsModule.forRoot([]),    !environment.production ? StoreDevtoolsModule.instrument() : [],    StoreRouterConnectingModule  ],  declarations: [AppComponent],  bootstrap: [AppComponent]})export class AppModule {}​
```

 Note: StoreFreeze does not currently work with out extra steps. For this workshop we will disable by replacing the lines below.

Issue with storeFreeze to be fixed in NgRx v6 https://github.com/nrwl/nx/issues/436

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

{% hint style="info" %}
 Note: StoreFreeze does not currently work with out extra steps. For this workshop we will disable by replacing the lines below.

Issue with storeFreeze to be fixed in NgRx v6 https://github.com/nrwl/nx/issues/436
{% endhint %}



##   {#6-examine-angular-app-and-module-structure}

