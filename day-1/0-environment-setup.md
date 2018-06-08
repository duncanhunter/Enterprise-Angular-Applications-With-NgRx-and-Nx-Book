# 0 - Environment Setup

Dependency checklist:

1. node v8+
2. git
3. cli.angular.io
4. nrwl schematics
5. visual studio code
6. visual studio code extensions
7. chrome extension for redux dev tools

The main dependency for being able to make an Angular application is node version 8+. The latest stable version of node is best to get if you do not have it already installed.

## Check you have the right node and git

You can check your version of node by running the following command in the terminal.

```text
node -v
```

If you would like to use source control and check out completed work then it is recommended to have git installed on your machine. You can download git from[ https://git-scm.com/downloads ](https://git-scm.com/downloads%20)

You can check your version of node by running the following command in the terminal.

```text
git --version
```

If you do not have node installed or you are using a version lower than v4 then I you can get the latest stable version from [www.nodejs.org](https://github.com/duncanhunter/Enterprise-Angular-Applications-With-NgRx-and-Nx-Book/tree/d63a57a9f1ea36a7623cdf0746dd90b1406edaa2/www.nodejs.org).

## Install Angular CLI and Nx

We need to have both the Angular CLI and the nrwl schematics installed globally. Run the following commands.

```text
npm install -g @angular/cli
```

```text
npm install -g @nrwl/schematics
```

* Manually install NgRx schematics globally until issue resolved

```text
npm i @ngrx/schematics -g
```

{% embed data="{\"url\":\"https://github.com/nrwl/nx/issues/531\",\"type\":\"link\",\"title\":\"Creating a new workspace failed with error · Issue \#531 · nrwl/nx\",\"description\":\"Im setting up a new project and following the docs found here https://nrwl.io/nx/guide-nx-workspace When I run ng new myworkspacename --collection=@nrwl/schematics I get this error. Could not find ...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://github.com/fluidicon.png\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://avatars2.githubusercontent.com/u/1406482?s=400&v=4\",\"width\":400,\"height\":400,\"aspectRatio\":1}}" %}

## **Get Visual Studio Code**  

[**https://code.visualstudio.com/**](https://code.visualstudio.com/)

![](../.gitbook/assets/vscode.png)

## Get **Visual Studio Code**  Extensions

Angular Essentials: Everything you need for angular in an extension pack

Rainbow Brackets: Handy for many brackets when inlining observables

TSLint: Great linting in VSCode

{% embed data="{\"url\":\"https://marketplace.visualstudio.com/items?itemName=johnpapa.angular-essentials\",\"type\":\"link\",\"title\":\"Angular Essentials - Visual Studio Marketplace\",\"description\":\"Extension for Visual Studio Code - Essential extensions for Angular developers\",\"icon\":{\"type\":\"icon\",\"url\":\"https://marketplace.visualstudio.com/favicon.ico\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://johnpapa.gallerycdn.vsassets.io/extensions/johnpapa/angular-essentials/0.3.2/1508504847990/Microsoft.VisualStudio.Services.Icons.Default\",\"width\":128,\"height\":128,\"aspectRatio\":1}}" %}

{% embed data="{\"url\":\"https://marketplace.visualstudio.com/items?itemName=2gua.rainbow-brackets\",\"type\":\"link\",\"title\":\"Rainbow Brackets - Visual Studio Marketplace\",\"description\":\"Extension for Visual Studio Code - A rainbow brackets extension for VS Code.\",\"icon\":{\"type\":\"icon\",\"url\":\"https://marketplace.visualstudio.com/favicon.ico\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://2gua.gallerycdn.vsassets.io/extensions/2gua/rainbow-brackets/0.0.6/1474455607820/Microsoft.VisualStudio.Services.Icons.Default\",\"width\":128,\"height\":128,\"aspectRatio\":1}}" %}

{% embed data="{\"url\":\"https://marketplace.visualstudio.com/items?itemName=eg2.tslint\",\"type\":\"link\",\"title\":\"TSLint - Visual Studio Marketplace\",\"description\":\"Extension for Visual Studio Code - TSLint for Visual Studio Code\",\"icon\":{\"type\":\"icon\",\"url\":\"https://marketplace.visualstudio.com/favicon.ico\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://eg2.gallerycdn.vsassets.io/extensions/eg2/tslint/1.0.30/1527489705111/Microsoft.VisualStudio.Services.Icons.Default\",\"width\":120,\"height\":120,\"aspectRatio\":1}}" %}



![](../.gitbook/assets/2016-11-09_17-02-23.png)

### Optionally turn on **Visual Studio Code  auto save**

![](../.gitbook/assets/2017-07-25_21-00-24.jpg)



