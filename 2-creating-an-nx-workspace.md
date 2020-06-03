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

* Create a new app by installing the Angular schematic and running the below command in the terminal in a directory of your choice.

```text
npm install @nrwl/angular
nx generate @nrwl/angular:app customer-portal --routing
```

The resulting output should look like this:

```txt
? Which stylesheet format would you like to use? SASS(.scss)  [ http://sass-lang.com   ]
CREATE libs/auth/README.md (132 bytes)
CREATE libs/auth/tsconfig.lib.json (408 bytes)
CREATE libs/auth/tsconfig.lib.prod.json (97 bytes)
CREATE libs/auth/tslint.json (244 bytes)
CREATE libs/auth/src/index.ts (35 bytes)
CREATE libs/auth/src/lib/auth.module.ts (268 bytes)
CREATE libs/auth/src/lib/auth.module.spec.ts (338 bytes)
CREATE libs/auth/tsconfig.json (123 bytes)
CREATE libs/auth/jest.config.js (345 bytes)
CREATE libs/auth/tsconfig.spec.json (233 bytes)
CREATE libs/auth/src/test-setup.ts (30 bytes)
UPDATE workspace.json (20717 bytes)
UPDATE nx.json (1174 bytes)
UPDATE tsconfig.json (691 bytes)
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
