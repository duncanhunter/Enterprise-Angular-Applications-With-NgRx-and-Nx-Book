# End-to-end routing tests

When we ran the end-to-end tests last time, it was a single run with the following command:

```txt
nx e2e customer-portal-e2e
```

According to [the official docs](https://nx.dev/latest/angular/cypress/overview), *by default, Cypress will run in “headed” mode (you will see the tests executing in a new browser window). You will have the result of all the tests and errors (if any) in your terminal.*

*Passing --headless. You will see all the test results live in the terminal. Videos and screenshots will be available for debugging.  In headless mode your tests will be re-run every time you make a change to your application code. Screenshots and videos will be accessible in dist/apps/frontend/screenshots and dist/apps/frontend/videos.*

As yet, there will be no screenshots, but there should be a video of the passed tests.

```txt
nx e2e customer-portal-e2e --headless --watch
```

## Test login

We want to create two more tests for a successful login and an unsuccessful login.

Continuing to use the Page Object pattern that's already started in the Cypress implementation, we can add some other page objects to the app.po.ts file.

apps\customer-portal-e2e\src\support\app.po.ts

```js
export const getGreeting = () => cy.get('.title');
export const getLoginNameField = () => cy.get('.login-name');
export const getLoginNamePassword = () => cy.get('.login-password');
export const getLoginButton = () => cy.get('.login-button');
export const getRightNav = () => cy.get('.right-nav');
```

For this to work, you will have to add classes to each of those components, like this:

```html
<input class="login-name"
  matInput
  placeholder="username"
  type="text"
  formControlName="username"
/>
```

Although there are no styles for that class yet, it will work to give us an easy hook on that input.  Do the same for the other elements shown above.

You could also use an id, or any other valid css selector as seen in the [official docs for the get() function](https://docs.cypress.io/api/commands/get#Arguments).

Then, create a new test file:

apps\customer-portal-e2e\src\integration\login.spec.ts

```js
import {
  getLoginNameField,
  getLoginNamePassword,
  getLoginButton,
  getRightNav,
} from '../support/app.po';

describe('Login Tests', function () {
  it('Successful login', function () {
    cy.visit('/auth/login');
    getLoginNameField().type('duncan');
    getLoginNamePassword().type('123');
    getLoginButton().click();
    getRightNav().contains('Products');
  });
});
```

Note that only the application tests are watched for changes.  You will have to do something like open the login-form.component.html and just save it again to re-run the e2e tests.
