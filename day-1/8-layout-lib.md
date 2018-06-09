---
description: >-
  In this section me make a reusable Layout and a BehaviorSubject to share User
  state
---

# 8 - Layout Lib and BehaviorSubjects



## ????. Share User state with a BehaviorSubject

A BehaviourSubject is a special observable you can both subscribe to and pass values.

* Add a BehaviorSubject to the Auth service.

{% code-tabs %}
{% code-tabs-item title="libs/auth/src/lib/services/auth/auth.service.ts" %}
```typescript
import { Injectable } from '@angular/core';
import { Authenticate, User } from '@demo-app/data-models';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private userSubject$ = new BehaviorSubject<User>(null);
  user$ = this.userSubject$.asObservable();

  constructor(private httpClient: HttpClient) {}

  login(authenticate: Authenticate): Observable<User> {
    return this.httpClient.post<User>(
      'http://localhost:3000/login',
      authenticate
    ).pipe(tap((user: User) => (this.userSubject$.next(user))));
  }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

