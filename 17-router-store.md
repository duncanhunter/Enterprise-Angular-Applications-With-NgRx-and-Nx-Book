---
description: In this section we examine the router store library from NgRx
---

# 17 - Router Store

## 1. Add a new presentational components for products

```text
nx generate @nrwl/angular:component containers/product-list --project=products
CREATE libs/products/src/lib/containers/product-list/product-list.component.html
CREATE libs/products/src/lib/containers/product-list/product-list.component.spec.ts
CREATE libs/products/src/lib/containers/product-list/product-list.component.ts
CREATE libs/products/src/lib/containers/product-list/product-list.component.scss
```

## 2. Add material module to Products module

{% code title="libs/products/src/lib/products.module.ts" %}

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { ProductsComponent } from './containers/products/products.component';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import {
  productsReducer,
  initialState as productsInitialState,
} from './+state/products.reducer';
import { ProductsEffects } from './+state/products.effects';
import { ProductListComponent } from './containers/product-list/product-list.component';
import { MaterialModule } from '@demo-app/material';
@NgModule({
  imports: [
    CommonModule,
    MaterialModule,
    RouterModule.forChild([{ path: '', component: ProductsComponent }]),
    StoreModule.forFeature('products', productsReducer, {
      initialState: productsInitialState,
    }),
    EffectsModule.forFeature([ProductsEffects]),
  ],
  declarations: [ProductsComponent, ProductListComponent],
  providers: [ProductsEffects],
})

export class ProductsModule {}
```

## 3. Add presentational list component onto the container components template

```html
<div fxLayout="row"  fxFlexLayout="center center">
  <demo-app-product-list [products]="products$ | async"></demo-app-product-list>
</div>
```

## 4. Update presentational ProductList component to receive products as inputs

{% code title="libs/products/src/lib/components/product-list/product-list.component.ts" %}

```typescript
import { Component, Input, EventEmitter } from '@angular/core';
import { Product } from '@demo-app/data-models';

@Component({
  selector: 'demo-app-product-list',
  templateUrl: './product-list.component.html',
  styleUrls: ['./product-list.component.scss'],
})
export class ProductListComponent {
  @Input() products: Product[];

  onFilter(category: string) {
    this.filter.emit(category);
  }
}
```

{% endcode %}

## 5. Add a material nav list to render a list of Products

{% code title="libs/products/src/lib/components/product-list/product-list.component.ts" %}

```html
<mat-card>
  <mat-form-field>
    <mat-select placeholder="Category" (selectionChange)="onFilter($event.value)">
      <mat-option [value]="'tools'">Tools</mat-option>
      <mat-option [value]="'paint'">Paint</mat-option>
    </mat-select>
  </mat-form-field>
  <mat-nav-list>
    <a mat-list-item *ngFor="let product of products">{{product.name}}</a>
  </mat-nav-list>
</mat-card>
```

{% endcode %}

## 6. Add material select to filter on category

* Add on filter method to component to emit a filter

{% code title="libs/products/src/lib/components/product-list/product-list.component.ts" %}

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { Product } from '@demo-app/data-models';

@Component({
  selector: 'demo-app-product-list',
  templateUrl: './product-list.component.html',
  styleUrls: ['./product-list.component.scss'],
})
export class ProductListComponent {
  @Input() products: Product[];
  @Output() filter = new EventEmitter<string>();

  onFilter(category: string) {
    this.filter.emit(category);
  }
}
```

{% endcode %}

* Add filter listener to Products container components template

{% code title="libs/products/src/lib/containers/products/products.component.html" %}

```html
<div fxLayout="row" fxFlexLayout="center center">
  <demo-app-product-list
    [products]="products$ | async"
    (filter)="updateUrlFilters($event)"
  ></demo-app-product-list>
</div>
```

{% endcode %}

## 7. Update URL on filter change and no not dispatch a LoadProducts action from the Products component

{% code title="lib\\containers\\products\\products.component.ts" %}

```typescript
import { Component, OnInit } from '@angular/core';
import { ProductsState } from './../../+state/products.reducer';
import { Store, select } from '@ngrx/store';
import { Observable } from 'rxjs';
import { Product } from '@demo-app/data-models';
import * as ProductsActions from '../../+state/products.actions';
import { productsQuery } from '../../+state/products.selectors';
import { NavigationExtras, Router } from '@angular/router';

@Component({
  selector: 'demo-app-products',
  templateUrl: './products.component.html',
  styleUrls: ['./products.component.scss'],
})
export class ProductsComponent implements OnInit {
  products$: Observable<Product[]>;
  selectedProduct$: Observable<Product>;

  constructor(private store: Store<ProductsState>, private router: Router) {}

  ngOnInit() {
    this.store.dispatch(ProductsActions.loadProducts());
    this.products$ = this.store.pipe(select(productsQuery.getProducts));
    this.selectedProduct$ = this.store.pipe(
      select(productsQuery.getSelectedProduct)
    );
  }

  updateUrlFilters(category: string): void {
    const navigationExtras: NavigationExtras = {
      replaceUrl: true,
      queryParams: { category },
    };
    this.router.navigate([`/products`], navigationExtras);
  }
}
```

{% endcode %}

## 8. Add an effect to listen for router actions

{% code title="libs\\products\\src\\lib\\+state\\products.effects.ts" %}

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { ProductsService } from './../services/products/products.service';
import { ProductsActionTypes } from './../+state/products.actions';
import { mergeMap, map, catchError, filter } from 'rxjs/operators';
import * as ProductActions from './../+state/products.actions';
import { of } from 'rxjs';
import { Product } from '@demo-app/data-models';
import { ROUTER_NAVIGATION, RouterNavigationAction } from '@ngrx/router-store';

@Injectable()
export class ProductsEffects {
  login$ = createEffect(() =>
    this.actions$.pipe(
      ofType(ProductsActionTypes.LoadProducts),
      mergeMap(() =>
        this.productService.getProducts().pipe(
          map((products: Product[]) =>
            ProductActions.loadProductsSuccess({ payload: products })
          ),
          catchError((error) =>
            of(ProductActions.loadProductsFailure({ error }))
          )
        )
      )
    )
  );

  
  loadFilteredProducts$ = createEffect(() => this.actions$.pipe(
    ofType(ROUTER_NAVIGATION),
    filter((r: RouterNavigationAction) =>
      r.payload.routerState.url.startsWith('/products')
    ),
    map(
      (r: RouterNavigationAction) =>
        r.payload.routerState.root.queryParams['category']
    ),
    mergeMap((category: string) =>
      this.productService.getProducts(category).pipe(
        map(
          (products: Product[]) =>
            ProductActions.loadProductsSuccess({ payload: products })
        ),
        catchError((error) => of(ProductActions.loadProductsFailure(error)))
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

## 9. Update the Products service to use the new params of category

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

  getProducts(category = null): Observable<Product[]> {
    const url =
      category !== null
        ? `http://localhost:3000/products?category=${category}`
        : `http://localhost:3000/products`;

    return this.httpClient.get<Product[]>(url);
  }
}
```

{% endcode %}
