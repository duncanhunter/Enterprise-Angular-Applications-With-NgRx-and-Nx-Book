---
description: >-
  In this section we challenge your understanding by adding a Products module
  like we did for login
---

# 15 - Add Products NgRx Feature Module

Steps

1. Add NgRx Products lib making it a state state  
2. Add Products Action Creators  
3. Add default state and interface  
4. Make new Product interface  
5. Make new ProductsService in products module  
6. Add effect  
7. Add reducer logic  
8. Dispatch and select products from the store into container component  
9. Dump products as JSON onto the page

## 1. Add NgRx Products lib making it a state state

```text
nx g @nrwl/angular:ngrx --module=libs/products/src/lib/products.module.ts --minimal false
```

## 2. Add Products Action Creators

{% code title="libs/products/src/lib/+state/products.actions.ts" %}

```typescript
import { createAction, props } from '@ngrx/store';
import { Product } from '@demo-app/data-models';

export enum ProductsActionTypes {
  LoadProducts = '[Products Page] Load Products',
  LoadProductsSuccess = '[Products API] Load Products Success',
  LoadProductsFail = '[Products API] LoadProducts Fail',
}

export const loadProducts = createAction(
  ProductsActionTypes.LoadProducts
);

export const loadProductsSuccess = createAction(
  ProductsActionTypes.LoadProductsSuccess,
  props<{ payload: Product [] }>()
);

export const loadProductsFailure = createAction(
  ProductsActionTypes.LoadProductsFail,
  props<{ error: any }>()
);
```

{% endcode %}

## 3. Add default state and interface

* Update state interface

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}

```typescript
import { createReducer, on, Action } from '@ngrx/store';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import { ProductsEntity } from './products.models';
import * as ProductsActions from './products.actions';
import { Product } from '@demo-app/data-models';

export const PRODUCTS_FEATURE_KEY = 'products';

/**
 * Interface for the 'Products' data used in
 *  - ProductsState, and
 *  - productsReducer
 */
export interface ProductsData {
  loading: boolean;
  products: Product[];
  error: '';
}

/**
 * Interface to the part of the Store containing ProductsState
 * and other information related to ProductsData.
 */
export interface ProductsState {
  readonly products: ProductsData;
}

export interface State extends EntityState<ProductsEntity> {
  selectedId?: string | number;
  loaded: boolean;
  error?: string | null;
}

export interface ProductsPartialState {
  readonly [PRODUCTS_FEATURE_KEY]: State;
}

export const productsAdapter: EntityAdapter<ProductsEntity> = createEntityAdapter<ProductsEntity>();

export const initialState: State = productsAdapter.getInitialState({
  action: ProductsActions,
  loaded: false,
});

export const productsReducer = createReducer(
  initialState,
  on(ProductsActions.loadProducts, (state) => ({
    ...state,
    loaded: false,
    error: null,
  })),
  on(ProductsActions.loadProductsSuccess, (state, { payload: products }) => ({
    ...state,
    products: products,
    loaded: true,
  })),
  on(ProductsActions.loadProductsFailure, (state, { error }) => ({
    ...state,
    error,
  }))
);

export function reducer(state: State | undefined, action: Action) {
  return productsReducer(state, action);
}
```

{% endcode %}

## 4. Make new Product interface

{% code title="libs/data-models/product.d.ts" %}

```typescript
export interface Product {
  name: string;
  category: string;
  id: number;
}
```

{% endcode %}

{% code title="libs/data-models/index.ts" %}

```typescript
export * from './lib/data-models.module';
```

{% endcode %}

## 5. Make new ProductsService in products module

When the CLI asks for the name of the service, add the path for it, and it will be created in the right place since we also provide the project in the command:

```text
> nx generate @nrwl/angular:service --project=products
√ What name would you like to use for the service? · services/products/products
CREATE libs/products/src/lib/products.service.spec.ts
CREATE libs/products/src/lib/products.service.ts
```

{% code title="libs/products/src/lib/services/products/products.service.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Product } from '@demo-app/data-models';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ProductsService {
  constructor(private httpClient: HttpClient) {}

  getProducts(): Observable<Product[]> {
    return this.httpClient.get<Product[]>('http://localhost:3000/products');
  }
}
```

{% endcode %}

## 6. Add effect

{% code title="libs/products/src/lib/+state/products.effects.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { ProductsService } from './../services/products/products.service';
import { ProductsActionTypes } from './../+state/products.actions';
import { mergeMap, map, catchError } from 'rxjs/operators';
import * as ProductActions from './../+state/products.actions';
import { of } from 'rxjs';
import { Product } from '@demo-app/data-models';

@Injectable()
export class ProductsEffects {
  
  login$ = createEffect(() => this.actions$.pipe(
    ofType(ProductsActionTypes.LoadProducts),
    mergeMap(() =>
      this.productService.getProducts().pipe(
        map(
          (products: Product[]) =>
            ProductActions.loadProductsSuccess({ payload: products })
        ),
        catchError((error) => of(ProductActions.loadProductsFailure({ error })))
      )
    )
  ));

  constructor(
    private actions$: Actions,
    private productService: ProductsService
  ) {}
}
```

{% endcode %}

## 7. Add reducer logic

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}

```typescript
import { createReducer, on, Action } from '@ngrx/store';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import { ProductsEntity } from './products.models';
import * as ProductsActions from './products.actions';
import { Product } from '@demo-app/data-models';

export const PRODUCTS_FEATURE_KEY = 'products';

/**
 * Interface for the 'Products' data used in
 *  - ProductsState, and
 *  - productsReducer
 */
export interface ProductsData {
  loading: boolean;
  products: Product[];
  error: '';
}

/**
 * Interface to the part of the Store containing ProductsState
 * and other information related to ProductsData.
 */
export interface ProductsState {
  readonly products: ProductsData;
}

export interface State extends EntityState<ProductsEntity> {
  selectedId?: string | number;
  loaded: boolean;
  error?: string | null;
}

export interface ProductsPartialState {
  readonly [PRODUCTS_FEATURE_KEY]: State;
}

export const productsAdapter: EntityAdapter<ProductsEntity> = createEntityAdapter<ProductsEntity>();

export const initialState: State = productsAdapter.getInitialState({
  action: ProductsActions,
  loaded: false,
});

export const productsReducer = createReducer(
  initialState,
  on(ProductsActions.loadProducts, (state) => ({
    ...state,
    loaded: false,
    error: null,
  })),
  on(ProductsActions.loadProductsSuccess, (state, { payload: products }) => ({
    ...state,
    products: products,
    loaded: true,
  })),
  on(ProductsActions.loadProductsFailure, (state, { error }) => ({
    ...state,
    error,
  }))
);

export function reducer(state: State | undefined, action: Action) {
  return productsReducer(state, action);
}
```

{% endcode %}

## 8. Dispatch and select products from the store into container component

{% code title="libs/products/src/lib/containers/products/products.component.ts" %}

```typescript
import { Component, OnInit } from '@angular/core';
import { ProductsState } from './../../+state/products.reducer';
import { Store, select } from '@ngrx/store';
import { productsQuery } from './../../+state/products.selectors';
import { Observable } from 'rxjs';
import { Product } from '@demo-app/data-models';
import { loadProducts } from './../../+state/products.actions';

@Component({
  selector: 'demo-app-products',
  templateUrl: './products.component.html',
  styleUrls: ['./products.component.scss'],
})
export class ProductsComponent implements OnInit {
  products$: Observable<Product[]>;

  constructor(private store: Store<ProductsState>) {}

  ngOnInit() {
    this.store.dispatch(loadProducts());
    this.products$ = this.store.pipe(select(productsQuery.getProducts));
  }
}
```

{% endcode %}

## 9. Dump products as JSON onto the page

{% code title="libs/products/src/lib/containers/products/products.component.html" %}

```markup
<p>
  products works!
</p>
{{products$ | async | json}}
```

{% endcode %}
