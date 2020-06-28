# 10 - NgRx Introduction

## 1. Presentation

Presentation: NgRx

[https://docs.google.com/presentation/d/1MknPww1MdwzreFvDh086TzqaB5yyQga0PaoRR9bgCXs/edit?usp=sharing](https://docs.google.com/presentation/d/1MknPww1MdwzreFvDh086TzqaB5yyQga0PaoRR9bgCXs/edit?usp=sharing)

## 2. Install redux dev tools chrome extension

[https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)

![](.gitbook/assets/redux-dev-tools.png)

## 3.  Simple Counter Demo

### a\) Make a new empty Angular CLI Project

We will make a simple NgRx demo app first before getting deep into it with Nx which will scaffold out an opinionated abstracted version.

```text
cd workshop
```

```text
ng new ngrx-counter-demo
```

```text
cd ngrx-counter-demo
```

```text
code .
```

### b\) Add NgRx to the app

```text
npm install @ngrx/store @ngrx/store-devtools
```

### c\) Add a counter.reducer.ts fill to a new state folder in the app

{% code title="src/app/state/counter.reducer.ts" %}

```typescript
export function counterReducer(state, action) {
  on('INCREMENT', state => state + 1),
  on('DECREMENT', state => state - 1),
  on(reset, state => 0),
);

```

{% endcode %}

### d\) Register the reducer and NgRx in the App module

{% code title="src/app/app.module.ts" %}

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { StoreModule } from '@ngrx/store';
import { counterReducer } from './state/counter.reducer';
import { StoreDevtoolsModule} from '@ngrx/store-devtools';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    StoreModule.forRoot({ counter: counterReducer }),
    StoreDevtoolsModule.instrument({})
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

{% endcode %}

### e\) Add the HTML to the App component

{% code title="src/app/app.component.html" %}

```markup
<button (click)="increment()">Increment</button>
<div>Current Count: {{ count$ | async }}</div>
<button (click)="decrement()">Decrement</button>
```

{% endcode %}

### f\) Add App component logic

{% hint style="info" %}
Note:  changeDetection: ChangeDetectionStrategy.OnPush to avoid issue with change detection.
{% endhint %}

{% code title="src/app/app.component.ts" %}

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';
import { select, Store } from '@ngrx/store';
import { Observable } from 'rxjs';
​
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  count$: Observable<number>;
​
  constructor(private store: Store<any>) {
    this.count$ = store.pipe(select('counter'));
  }
​
  increment() {
    this.store.dispatch({ type: 'INCREMENT' });
  }
​
  decrement() {
    this.store.dispatch({ type: 'DECREMENT' });
  }
​
}
```

{% endcode %}

### g\) Adding action creators

At this point we have no strong types for our actions or our state tree, let's fix that.

* Add a file to the state folder called counter.actions.ts

{% code title="src/app/state/counter.actions.ts" %}

```typescript
import { Action } from '@ngrx/store';

export enum CounterActionTypes {
    Increment = '[Counter Page] Increment',
    Decrement = '[Counter Page] Decrement'
}

export const increment = createAction(CounterActionTypes.Increment);

export const decrement = createAction(CounterActionTypes.Decrement);
```

{% endcode %}

### h\) Adding action State interfaces

* In the top of the reducer file at a Product State interface and use it and our new action types int he reducer.

{% code title="src/app/state/counter.reducer.ts" %}

```typescript
import { CounterActions, CounterActionTypes } from 'src/app/state/counter.actions';

export interface CounterState {
  count: number;
}

export const initialState: CounterState = {
  count: 0
};

export function counterReducer(state = initialState, action: CounterActions) {
  switch (action.type) {
    case CounterActionTypes.Increment:
      return {
        count: state.count + 1
      };

    case CounterActionTypes.Decrement:
      return {
        count: state.count - 1
      };

    default:
      return state;
  }
}

```

{% endcode %}

### i\) Add global State interface

This is useful to be able to type your injected store and select state.

{% code title="src/app/state/appState.ts" %}

```typescript
import { CounterState } from 'src/app/state/counter.reducer';

export interface AppState {
    counter: CounterState;
}
```

{% endcode %}

### j\) Use state interface in app component

{% code title="src/app/app.component.ts" %}

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';
import { select, Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { Increment, Decrement } from 'src/app/state/counter.actions';
import { AppState } from 'src/app/state/appState';                 // Added

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  count$: Observable<number>;
​
  constructor(private store: Store<AppState>) {                      // Added
    this.count$ = store.pipe(select(state => state.counter.count));  // Added
  }
​
  increment() {
    this.store.dispatch(new Increment());
  }
​
  decrement() {
    this.store.dispatch(new Decrement());
  }
​
}

```

{% endcode %}
