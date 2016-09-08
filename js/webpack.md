# Webpack

## Webpack 2
### Setting up Tree shaking for ES6 modules

Webpack 2 is now supporting the ES6 module syntax
```js
import { Thing } from 'my-module'
```

There is no need to transpile `import` to `require()` calls anymore!

However we still need to transpile code from ES6+ to ES5 using Babel.
In .babelrc, using the latest version of  `babel-preset-es2015`, you just have to change your config from :

```
{
  "presets": ["es2015", ...]
}
```

to

```
{
  "presets": [["es2015", {modules: false}], ...]
}
```

And Babel will stop transpiling `import` to `require()` 😉
