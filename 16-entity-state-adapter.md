---
description: In this section we add entity adapter
---

# 16 - Entity State Adapter

##  1. npm install entity from NgRx

```text
npm install @ngrx/entity
```

## 2. Use entity adapter in reducer

* Extend ProductState with EntityState. By default it will make an entities and ids dictionary. You Add to this any state properties you desire.
* Create an adapter and use it 's getInitialState method to make the initial state

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}
```typescript
import { Action } from '@ngrx/store';
import { ProductsActions, ProductsActionTypes } from './products.actions';
import { Product } from '@demo-app/data-models';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';

export interface ProductsData extends EntityState<Product> {
  error: string;
  selectedProductId: number;
  loading: boolean;
}

export interface ProductsState {
  readonly products: ProductsData;
}

export const adapter: EntityAdapter<Product> = createEntityAdapter<Product>({});

export const initialState: ProductsData = adapter.getInitialState({
  error: '',
  selectedProductId: null,
  loading: false
});

/// Abbreviated
```
{% endcode %}

## 3. Update the default reducer logic

Adapter Collection Methods

The entity adapter also provides methods for operations against an entity. These methods can change one to many records at a time. Each method returns the newly modified state if changes were made and the same state if no changes were made.

* `addOne`: Add one entity to the collection
* `addMany`: Add multiple entities to the collection
* `addAll`: Replace current collection with provided collection
* `removeOne`: Remove one entity from the collection
* `removeMany`: Remove multiple entities from the collection
* `removeAll`: Clear entity collection
* `updateOne`: Update one entity in the collection
* `updateMany`: Update multiple entities in the collection
* `upsertOne`: Add or Update one entity in the collection
* `upsertMany`: Add or Update multiple entities in the collection

[https://github.com/ngrx/platform/blob/master/docs/entity/adapter.md](https://github.com/ngrx/platform/blob/master/docs/entity/adapter.md)

{% code title="libs/admin-portal/users/+state/users.reducer.ts" %}
```typescript
export function productsReducer(
  state = initialState,
  action: ProductsActions
): ProductsData {
  switch (action.type) {
    case ProductsActionTypes.LoadProducts:
      return { ...state, loading: true };

    case ProductsActionTypes.LoadProductsSuccess: {
      return adapter.addAll(action.payload, { ...state, error: '' });
    }

    case ProductsActionTypes.LoadProductsFail: {
      return adapter.removeAll({ ...state, error: action.payload });
    }

    default:
      return state;
  }
}
```
{% endcode %}

## 4. Add selector methods to bottom of reducer

```text
export const getSelectedProductId = (state: ProductsData) =>
  state.selectedProductId;

export const {
  // select the array of user ids
  selectIds: selectProductIds,

  // select the dictionary of Products entities
  selectEntities: selectProductEntities,

  // select the array of Productss
  selectAll: selectAllProducts,

  // select the total Products count
  selectTotal: selectProductsTotal
} = adapter.getSelectors();
```

## 5. Add default selectors to use entity adapter

{% code title="libs/products/src/lib/+state/products.selectors.ts" %}
```typescript
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { ProductsState, ProductsData } from './products.reducer';
import * as fromProduct from './products.reducer';

const getProductsState = createFeatureSelector<ProductsData>('products');

const getProducts = createSelector(
  getProductsState,
  fromProduct.selectAllProducts
);
const getProductEntnites = createSelector(
  getProductsState,
  fromProduct.selectProductEntities
);
const getSelectedProductId = createSelector(
  getProductsState,
  fromProduct.getSelectedProductId
);
const getSelectedProduct = createSelector(
  getProductEntnites,
  getSelectedProductId,
  (productsDictionary, id) => {
    return productsDictionary[id];
  }
);

export const productsQuery = {
  getProducts,
  getProductEntnites,
  getSelectedProductId,
  getSelectedProduct
};

```
{% endcode %}

## 6. Examine State tree in Devtools

![Entities and Ids vs Arrays in the State Tree](.gitbook/assets/image%20%2824%29.png)

