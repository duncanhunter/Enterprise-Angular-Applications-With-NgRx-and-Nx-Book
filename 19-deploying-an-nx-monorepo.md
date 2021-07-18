# 18 - Deploying An Nx Monorepo

The AngularCLI only generates bundles. This means that we cannot build the lib itself. We can only do it by building an app that depends on it.

And then run

```text
nx build --prod customer-portal
```

In this section, we will run some analysis tools to inspect the size of our application.

* Install the [webpack-bundle-analyzer](https://github.com/th0r/webpack-bundle-analyzer)

```text
npm install  --save-dev webpack-bundle-analyzer
```

* Once installed add the following entry to the npm scripts in the package.json:

{% code title="package.json" %}

```bash
"bundle-report-customer-portal": "webpack-bundle-analyzer dist/apps/customer-portal/stats.json"
```

{% endcode %}

* Rebuild with --stats-json

```text
nx build --prod customer-portal --stats-json
```

* Run the following command

```text
npm run bundle-report-customer-portal
```

* Run the Nx dep graph tool

```text
nx dep-graph
```
