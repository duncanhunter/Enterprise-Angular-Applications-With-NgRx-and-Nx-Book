---
description: >-
  In this section we challenge you understanding by adding a Products module
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
ng generate ngrx products --module=libs/products/src/lib/products.module.ts
```

## 2. Add Products Action Creators

{% code title="libs/products/src/lib/+state/products.actions.ts" %}
```typescript
import { Action } from '@ngrx/store';

export enum ProductsActionTypes {
  LoadProducts = '[Products Page] Load Products',
  LoadProductsSuccess = '[Products API] Load Products Success',
  LoadProductsFail = '[Products API] LoadProducts Fail'
}

export class LoadProducts implements Action {
  readonly type = ProductsActionTypes.LoadProducts;
}
export class LoadProductsSuccess implements Action {
  readonly type = ProductsActionTypes.LoadProductsSuccess;
  constructor(public payload: any) {}
}

export class LoadProductsFail implements Action {
  readonly type = ProductsActionTypes.LoadProductsFail;
  constructor(public payload: any) {}
}

export type ProductsActions = LoadProducts | LoadProductsSuccess | LoadProductsFail;

```
{% endcode %}

## 3. Add default state and interface

* Update state interface

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}
```typescript
import { ProductsActions, ProductsActionTypes } from './products.actions';
import { Product } from '@demo-app/data-models';

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

export const initialState: ProductsData = {
  loading: false,
  products: [],
  error: ''
};
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
export { Authenticate } from './authenticate';
export { User } from './user';
export { Product } from './product';
```
{% endcode %}

## 5. Make new ProductsService in products module

```text
ng g service services/products/products --project=products
```

{% code title="libs/products/src/lib/services/products/products.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Product } from '@demo-app/data-models';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
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
import { Actions, Effect, ofType } from '@ngrx/effects';
import { ProductsService } from './../services/products/products.service';
import { ProductsActionTypes } from './../+state/products.actions';
import { mergeMap, map, tap, catchError } from 'rxjs/operators';
import * as productActions from './../+state/products.actions';
import { of } from 'rxjs';
import { Product } from '@demo-app/data-models';

@Injectable()
export class ProductsEffects {
  @Effect()
  login$ = this.actions$.pipe(
    ofType(ProductsActionTypes.LoadProducts),
    mergeMap(() =>
      this.productService
        .getProducts()
        .pipe(
          map(
            (products: Product[]) =>
              new productActions.LoadProductsSuccess(products)
          ),
          catchError(error => of(new productActions.LoadProductsFail(error)))
        )
    )
  );

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
import { Action } from '@ngrx/store';
import { ProductsActions, ProductsActionTypes } from './products.actions';
import { Product } from '@demo-app/data-models';

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

export const initialState: ProductsData = {
  loading: false,
  products: [],
  error: ''
};

export function productsReducer(
  state = initialState,
  action: ProductsActions
): ProductsData {
  switch (action.type) {
    case ProductsActionTypes.LoadProducts:
      return { ...state, loading: true };

    case ProductsActionTypes.LoadProductsSuccess: {
      return { ...state, products: action.payload, loading: false, error: '' };
    }
    case ProductsActionTypes.LoadProductsFail: {
      return {
        ...state,
        products: null,
        loading: false,
        error: action.payload
      };
    }
    default:
      return state;
  }
}

```
{% endcode %}

## 8. Dispatch and select products from the store into container component

{% code title="libs/products/src/lib/containers/products/products.component.ts" %}
```typescript
import { Component, OnInit } from '@angular/core';
import { ProductsState } from './../../+state/products.reducer';
import { Store, select } from '@ngrx/store';
import { productsQuery } from './../../+state';
import { Observable } from 'rxjs';
import { Product } from '@demo-app/data-models';
import { LoadProducts } from './../../+state/products.actions';

@Component({
  selector: 'demo-app-products',
  templateUrl: './products.component.html',
  styleUrls: ['./products.component.css']
})
export class ProductsComponent implements OnInit {
  products$: Observable<Product[]>;

  constructor(private store: Store<ProductsState>) { }

  ngOnInit() {
    this.store.dispatch(new LoadProducts());
    this.products$ = this.store.pipe(select(productsQuery.getProducts))
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

