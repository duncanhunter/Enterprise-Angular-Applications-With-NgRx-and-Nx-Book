---
description: Now it is time to talk Nx and architecture
---

# 2 -  Creating an Nx Workspace

We will be building out the beginning of two applications a customer portal and an admin portal.

![Nx workspaces diagram](../.gitbook/assets/image%20%286%29.png)

## 1 .Create a new Nx workspace

* Make a new folder for your work space and change directory into it.

```text
mkdir demo-app
cd demo-app
```

* Run the below command in a terminal to make a new nx workspace.

```text
ng new demo-app --collection=@nrwl/schematics
```

## 2. Examine the output of the following files and commit code to git source control

Run the following command to open the new nx workspace in VSCode.

```text
cd code .
```

* Inspect the following files:
  * angular.json
  * nx.json
  * tsconfig paths
  * package.json

## 3. Create a new app

* By default a new NX workspace has no apps or libs yet. You can run the below command to see the extra options to make an app or a lib besides the normal angular CLI commands.

```text
ng g application --help
```

* Create a new app by running the below command in the terminal in a directory of your choice. 

```text
ng g application customer-portal --style=scss --routing --prefix=app
```

## 4. Examine angular app and module structure

* app.module.ts
* tsconfig paths
* package.json
* apps and libs folders are empty

## 5. Run the app in the Chrome browser

* Run the following command to launch the app in the browser. -a is for the app to start.

```text
ng s --project=customer-portal
```

## 6. Commit your code

* It is best to commit often when using the code generation tools so you can easily undo if it is not correct.

![VS Codes Source Control Panel](../.gitbook/assets/image%20%283%29.png)

## 7. Install WIP Workshop Code in a different folder {#wipcoursecode}

```text
git clone "https://github.com/duncanhunter/WIP-Demo-App-NDC-Oslo-2018-Enterprise-Angular-applications-with-ngrx-and-nx.git"
​
cd WIP-Demo-App-NDC-Oslo-2018-Enterprise-Angular-applications-with-ngrx-and-nx

npm install
```

