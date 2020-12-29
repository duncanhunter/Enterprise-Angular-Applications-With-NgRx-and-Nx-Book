---
description: In this section we examine using selectors in NgRx.
---

# 14 - NgRx Selectors

## 1. Add selector file

* Add a file called index.ts to the +state folder of your auth state lib

{% code title="libs/auth/src/lib/+state/products.selectors.ts" %}

```typescript
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { ProductsState, ProductsData } from './products.reducer';
import * as fromProduct from './products.reducer';

const getProductsState = createFeatureSelector<ProductsData>('products');

const getProducts = createSelector(
  getProductsState,
  state => state.products
);

export const productsQuery = {
  getProducts,
};

```

{% endcode %}

* Ensure you have re-exported your publicly available paths in the auth libs index.ts file

{% code title="libs/auth/index.ts" %}

```typescript
export * from './lib/auth.module';
export { AuthService } from './lib/services/auth/auth.service';
export { AuthGuard } from './lib/guards/auth/auth.guard';
export { AuthState } from './lib/+state/auth.reducer';
export * from './lib/+state';
```
{% endcode %}

## 2. Use selector in Layout component

{% code title="libs/layout/src/lib/containers/layout/layout.component.ts" %}

```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { AuthState } from '@demo-app/auth';
import { User } from '@demo-app/data-models';
import { Observable } from 'rxjs/Observable';
import { productsQuery } from '@demo-app/auth';
@Component({
  selector: 'app-layout',
  templateUrl: './layout.component.html',
  styleUrls: ['./layout.component.scss']
})
export class LayoutComponent implements OnInit {
  user$: Observable<User>;

  constructor(private store: Store<AuthState>) {}

  ngOnInit() {
    this.user$ = this.store.select(productsQuery.getUser);
  }
}

```

{% endcode %}
