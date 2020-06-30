# 9b - Route Guards unit tests

So far the auth module has not been tested.  Run the command and add it to the list:

```bash
nx test auth --watch
```

Since we haven't updated the dependencies of the automatically generated tests, the result is expected:

```bash
 FAIL  libs/auth/src/lib/guards/auth/auth.guard.spec.ts (6.572s)
  AuthGuard
    × should be created (362ms)
  ● AuthGuard › should be created
    NullInjectorError: StaticInjectorError(DynamicTestModule)[Router]:
      StaticInjectorError(Platform: core)[Router]:
        NullInjectorError: No provider for Router!
```

There are no arrays at all in the configureTestingModule({}) object.  Add it now:

```js
import { RouterTestingModule } from '@angular/router/testing';
...
 imports: [RouterTestingModule]
```

The next error is:

```bash
NullInjectorError: No provider for HtHttpClient
tpClient!
```

That requires the HttpClientTestingModule.

```js
import { HttpClientTestingModule } from '@angular/common/http/testing';
...
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [
        RouterTestingModule, 
        HttpClientTestingModule]
    });
    guard = TestBed.inject(AuthGuard);
  });
```

Now the tests pass.  It would be wise at this point to write a meaningful test to confirm the guard is working without having to test it manually.  This will give you some confidence that the app is still secure in the future after other developers start to add code to the app.

Consider this task as extra credit.
