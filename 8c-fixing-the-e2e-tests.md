# Fixing the end-to-end tests

This time we are going to be adding the end-to-end tests of the customer-portal app.

To run those, use this command: nx e2e customer-portal-e2e.

You will see an output like this:

```txt
>nx e2e customer-portal-e2e
> nx run customer-portal-e2e:e2e
- Generating browser application bundles...
√ Browser application bundle generation complete.
...
** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
√ Compiled successfully.
It looks like this is your first time using Cypress: 6.9.1
[11:44:08]  Verifying Cypress can run C:\Users\timof\AppData\Local\Cypress\Cache\6.9.1\Cypress [started]
[11:44:18]  Verified Cypress! C:\Users\timof\AppData\Local\Cypress\Cache\6.9.1\Cypress [title changed]
[11:44:18]  Verified Cypress! C:\Users\timof\AppData\Local\Cypress\Cache\6.9.1\Cypress [completed]
...
  customer-portal
    1) should display welcome message
  0 passing (5s)
  1 failing
  1) customer-portal
       should display welcome message:
     AssertionError: Timed out retrying after 4000ms: Expected to find element: `h1`, but never found it.
      at getGreeting (http://localhost:4200/__cypress/tests?p=src\integration\app.spec.ts:6:30)
```

The failing tests are in this file:

apps\customer-portal-e2e\src\integration\app.spec.ts

```js
import { getGreeting } from '../support/app.po';
describe('customer-portal', () => {
  beforeEach(() => cy.visit('/'));
  it('should display welcome message', () => {
    // Custom command example, see `../support/commands.ts` file
    cy.login('my-email@something.com', 'myPassword');
    // Function helper example, see `../support/app.po.ts` file
    getGreeting().contains('Welcome to customer-portal!');
  });
});
```

apps\customer-portal-e2e\src\support\app.po.ts

```js
export const getGreeting = () => cy.get('h1');
```

If we change 'h1' to '.title', adding a title class to the layout.component.html

```html
<span class="title">Customer Portal</span>
```

And update the string in the contains function of the app.spec.ts file like this:

```js
getGreeting().contains('Customer Portal');
```

Then that test will pass.

When login is working, we can come back here and make a test for that to show that the login routing works, etc.

Right now, the server reports either an "unknown error" if the server is not running, "Unauthorized" if the email/password are wrong, and a JSON response with a fictional user-token at the moment.

That will be updated in the next section, step 9 - Route Guards and Products Lib.
