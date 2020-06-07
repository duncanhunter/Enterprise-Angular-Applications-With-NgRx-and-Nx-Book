# 2 -  Creating an Nx Workspace

We will be building out the beginning of two applications a customer portal and an admin portal.

![Nx workspaces diagram](.gitbook/assets/image%20%287%29.png)

## 1 .Create a new Nx workspace in your workshop folder

* Run the below command in a terminal to make a new nx workspace.

```text
npx create-nx-workspace@latest
```

Answer the questions as shown to create a blank workspace named demo-app that uses the nx CLI.  The output will look something like this:

```txt
npx: installed 198 in 37.986s
? Workspace name (e.g., org name)     demo-app
? What to create in the new workspace empty             [an empty workspace]
? CLI to power the Nx workspace       Nx           [Extensible CLI for JavaScript and TypeScript applications]
Creating a sandbox with Nx...
[##################################################################################################################################################] 364/364 new demo-app --preset="empty" --interactive=false --collection=@nrwl/workspace
    Successfully initialized git.
CREATE demo-app/nx.json (463 bytes)
CREATE demo-app/tsconfig.json (509 bytes)
CREATE demo-app/package.json (1100 bytes)
CREATE demo-app/README.md (2538 bytes)
CREATE demo-app/.editorconfig (245 bytes)
CREATE demo-app/.gitignore (503 bytes)
CREATE demo-app/.prettierignore (74 bytes)
CREATE demo-app/.prettierrc (26 bytes)
CREATE demo-app/workspace.json (1130 bytes)
CREATE demo-app/apps/.gitkeep (1 bytes)
CREATE demo-app/libs/.gitkeep (0 bytes)
CREATE demo-app/tools/tsconfig.tools.json (218 bytes)
CREATE demo-app/tools/schematics/.gitkeep (0 bytes)
CREATE demo-app/.vscode/extensions.json (109 bytes)
```

## 2. Examine the output of the following files and commit code to git source control

Change directories into the newly created workspace and run the following command to open the new nx workspace in VSCode.

```text
cd demo-app
cd code .
```

* Inspect the following files:
  * workshop.json
  * nx.json
  * tsconfig paths
  * package.json

## 3. Create a new app

* Since the new NX workspace has no apps or libs yet. You can run the below command to see the extra options to make an app or a lib besides the normal angular CLI commands.

```text
nx g application --help
```

* Install the Angular schematic needed to create an Angular app.

```text
npm install @nrwl/angular
```

* Create a new app by running the below command in the terminal in a directory of your choice.  Choose [SASS](https://sass-lang.com/documentation) as your stylesheet language.

```txt
nx generate @nrwl/angular:app customer-portal --routing
```

```txt
>nx generate @nrwl/angular:app customer-portal --routing
? Which stylesheet format would you like to use? (Use arrow keys)
  CSS
> SASS(.scss)  [ http://sass-lang.com   ]
  Stylus(.styl)[ http://stylus-lang.com ]
  LESS         [ http://lesscss.org     ]
CREATE apps/customer-portal/tsconfig.json (97 bytes)
CREATE apps/customer-portal/src/favicon.ico (15086 bytes)
CREATE apps/customer-portal/browserslist (429 bytes)
CREATE apps/customer-portal/tsconfig.app.json (163 bytes)
CREATE apps/customer-portal/tslint.json (247 bytes)
CREATE apps/customer-portal/src/index.html (338 bytes)
CREATE apps/customer-portal/src/main.ts (377 bytes)
CREATE apps/customer-portal/src/polyfills.ts (2833 bytes)
CREATE apps/customer-portal/src/styles.scss (80 bytes)
CREATE apps/customer-portal/src/assets/.gitkeep (0 bytes)
CREATE apps/customer-portal/src/environments/environment.prod.ts (52 bytes)
CREATE apps/customer-portal/src/environments/environment.ts (663 bytes)
CREATE apps/customer-portal/src/app/app.module.ts (419 bytes)
CREATE apps/customer-portal/src/app/app.component.html (3017 bytes)
CREATE apps/customer-portal/src/app/app.component.spec.ts (1051 bytes)
CREATE apps/customer-portal/src/app/app.component.ts (226 bytes)
CREATE apps/customer-portal/src/app/app.component.scss (2088 bytes)
CREATE apps/customer-portal/jest.config.js (369 bytes)
CREATE apps/customer-portal/tsconfig.spec.json (233 bytes)
CREATE apps/customer-portal/src/test-setup.ts (30 bytes)
CREATE apps/customer-portal-e2e/tslint.json (97 bytes)
CREATE apps/customer-portal-e2e/cypress.json (430 bytes)
CREATE apps/customer-portal-e2e/tsconfig.e2e.json (188 bytes)
CREATE apps/customer-portal-e2e/tsconfig.json (137 bytes)
CREATE apps/customer-portal-e2e/src/fixtures/example.json (80 bytes)
CREATE apps/customer-portal-e2e/src/integration/app.spec.ts (422 bytes)
CREATE apps/customer-portal-e2e/src/plugins/index.js (832 bytes)
CREATE apps/customer-portal-e2e/src/support/app.po.ts (47 bytes)
CREATE apps/customer-portal-e2e/src/support/commands.ts (1068 bytes)
CREATE apps/customer-portal-e2e/src/support/index.ts (599 bytes)
UPDATE package.json (1958 bytes)
UPDATE workspace.json (9408 bytes)
UPDATE nx.json (750 bytes)
UPDATE tslint.json (2311 bytes)
```

## 4. Examine angular app and module structure

* app.module.ts
* tsconfig paths
* package.json
* apps and libs folders are empty

## 5. Run the app in the Chrome browser

* Run the following command to launch the app in the browser. -a is for the app to start.

```text
nx serve customer-portal
```

## 6. Commit your code

* It is best to commit often when using the code generation tools so you can easily undo if it is not correct.

```txt
git add .
git commit -m "generated customer-portal Angular app"
```

![VS Codes Source Control Panel](.gitbook/assets/image%20%283%29.png)

## 7. Install WIP Workshop Code in a different folder <a id="wipcoursecode"></a>

```text
git clone "https://github.com/duncanhunter/WIP-Demo-App-NDC-Oslo-2018-Enterprise-Angular-applications-with-ngrx-and-nx.git"
​
cd WIP-Demo-App-NDC-Oslo-2018-Enterprise-Angular-applications-with-ngrx-and-nx

npm install
```
