---
description: In this section we add entity adapter
---

# 16 - Entity State Adapter

## 1. npm install entity from NgRx

In the past you had to install the @ngrx/entity package separately.  Now, you might have noticed in step 13 when we generated our store with the @nrwl/angular:ngrx schematic, it was added automatically.  Here is a brief overview of the Entity package and why we use it.

The store is like an in-memory database, and the Entity class gives a unique identifier to each object similar to a primary key.  In this way objects can then be flattened out and linked together using the entity unique identifiers, just like in a database. This makes it simpler to combine the multiple entities and 'join' them via a selector query.

The Entity State format consists of the objects in a map called "entities", and store the order of the objects in an array of ids.

The Adapter is a utility class that provides a series of functions designed to make it really simple to manipulate the entity state, such as changing the order.

You can find out more by reading [this Angular University piece on the subject](https://blog.angular-university.io/ngrx-entity/).

## 2. Use entity adapter in reducer

* Extend ProductState with EntityState. By default it will make an entities and ids dictionary. You Add to this any state properties you desire.
* Create an adapter and use it's getInitialState method to make the initial state

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}

```typescript
import { createReducer, on, Action } from '@ngrx/store';
import { EntityState, EntityAdapter, createEntityAdapter } from '@ngrx/entity';
import * as ProductsActions from './products.actions';
import { Product } from '@demo-app/data-models';

export const PRODUCTS_FEATURE_KEY = 'products';

export interface ProductsData extends EntityState<Product> {
  selectedProductId?: string | number;
  loading: boolean;
  error?: string | null;
}

export interface ProductsState {
  readonly products: ProductsData;
}

export const productsAdapter: EntityAdapter<Product> = createEntityAdapter<Product>({});

export interface ProductsState {
  readonly products: ProductsData;
}

export const productsAdapter: EntityAdapter<Product> = createEntityAdapter<Product>({});

export const initialState: ProductsData = productsAdapter.getInitialState({
  error: '',
  selectedProductId: null,
  loading: false,
});
/// Abbreviated
```

{% endcode %}

## 3. Update the default reducer logic

Adapter Collection Methods

The entity adapter also provides methods for operations against an entity. These methods can change one to many records at a time. Each method returns the newly modified state if changes were made and the same state if no changes were made.

* `addOne`: Add one entity to the collection
* `addMany`: Add multiple entities to the collection
* `setAll`: Replace current collection with provided collection
* `setOne`: Add or Replace one entity in the collection.
* `setMany`: Add or Replace multiple entities in the collection
* `removeOne`: Remove one entity from the collection
* `removeMany`: Remove multiple entities from the collection
* `removeAll`: Clear entity collection
* `updateOne`: Update one entity in the collection
* `updateMany`: Update multiple entities in the collection
* `upsertOne`: Add or Update one entity in the collection
* `upsertMany`: Add or Update multiple entities in the collection
* `mapOne`: Update one entity in the collection by defining a map function
* `map`: Update multiple entities in the collection by defining a map function, similar to Array.map

[https://ngrx.io/guide/entity/adapter](https://ngrx.io/guide/entity/adapter)

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}

```typescript
export const productsReducer = createReducer(
  initialState,
  on(ProductsActions.loadProducts, (state) => ({
    ...state,
    loading: false,
  })),
  on(ProductsActions.loadProductsSuccess, (state, { payload: products }) => {
    return productsAdapter.setAll(products, state);
  }),
  on(ProductsActions.loadProductsFailure, (state, { error }) =>
    productsAdapter.removeAll({
      ...state,
      error,
    })
  )
);

export function reducer(state: State | undefined, action: Action) {
  return productsReducer(state, action);
}
```

{% endcode %}

## 4. Add selector methods to bottom of reducer

{% code title="libs/products/src/lib/+state/products.reducer.ts" %}

```ts
export const getSelectedProductId = (state: ProductsData) =>
  state.selectedProductId;

export const {
  // select the array of user ids
  selectIds: selectProductIds,

  // select the dictionary of Products entities
  selectEntities: selectProductEntities,

  // select the array of Products
  selectAll: selectAllProducts,

  // select the total Products count
  selectTotal: selectProductsTotal
} = productsAdapter.getSelectors();
```

## 5. Add default selectors to use entity adapter

{% code title="libs/products/src/lib/+state/products.selectors.ts" %}

```typescript
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { ProductsData } from './products.reducer';
import * as fromProduct from './products.reducer';

const getProductsState = createFeatureSelector<ProductsData>('products');

const getProducts = createSelector(
  getProductsState,
  fromProduct.selectAllProducts
);
const getProductEntities = createSelector(
  getProductsState,
  fromProduct.selectProductEntities
);
const getSelectedProductId = createSelector(
  getProductsState,
  fromProduct.getSelectedProductId
);
const getSelectedProduct = createSelector(
  getProductEntities,
  getSelectedProductId,
  (productsDictionary, id) => {
    return productsDictionary[id];
  }
);

export const productsQuery = {
  getProducts,
  getProductEntities,
  getSelectedProductId,
  getSelectedProduct,
};
```

{% endcode %}

## 6. Examine State tree in Devtools

![Entities and Ids vs Arrays in the State Tree](.gitbook/assets/image%20%2824%29.png)

At this point, if you run the app however, you might get an error like this:

```txt
Error: libs/products/src/lib/containers/products/products.component.ts:21:5 - error TS2322: Type 'Observable<ProductsEntity[]>' is not assignable to type 'Observable<Product[]>'.
  Type 'ProductsEntity[]' is not assignable to type 'Product[]'.
    Type 'ProductsEntity' is missing the following properties from type 'Product': name, category
21     this.products$ = this.store.pipe(select(productsQuery.getProducts));
```

The app is not going to run now without updating the products component.  This will be fixed in the next step.
