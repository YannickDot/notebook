# AngularJS 1.X


### Lazy loading route controllers and templates using ui-router and webpack code splitting

Code splitting using webpack is a really neat feature that is often overlooked.
A split point in the dependency graph can be created using `require.ensure()` calls.

Let say we have the following module configuration :


```js
// my-module.js

import {
  IndexCtrl,
  ListingCtrl,
  AboutCtrl
} from './controllers.js'

var myModule = angular.module('myModule', [])

const myConfig = ($stateProvider) => {

  $stateProvider
  .state('index', {
    url: '/',
    template: require('path/to/my/template.html'),
    controller: 'IndexCtrl as vm',
  })
  .state('listing', {
    url: '/listing',
    template: require('path/to/my/template.html'),
    controller: 'ListingCtrl as vm',
  })
  .state('about', {
    url: '/',
    template: require('path/to/my/template.html'),
    controller: 'AboutCtrl as vm',
  })

}

myModule.config(myConfig)
.controller('IndexCtrl', IndexCtrl)
.controller('ListingCtrl', ListingCtrl)
.controller('AboutCtrl', AboutCtrl)

export default myModule

```



We can see here that our controllers and our templates are registered on the module when the module is defined (i.e when the js code is parsed and executed by the browser).

If we want to split our bundle and fetch controllers and templates on demand, we need to ensure that its respective javascript code is loaded before registering them.

The best way to defer the controller registration on the module is by using a closure.

#### Preparing the field

```js
function registerControllers(module) {
  module.controller('IndexCtrl', IndexCtrl)
  module.controller('ListingCtrl', ListingCtrl)
  module.controller('AboutCtrl', AboutCtrl)
}
```

We need to pass an already existing module to `registerControllers` and it will register our controllers on it.


We now move `registerControllers` in `controllers.js` file and export it


```js
// controllers.js

export const IndexCtrl = () => {}
export const ListingCtrl = () => {}
export const AboutCtrl = () => {}

export const registerIndexCtrl = (module) => {
  module.controller('IndexCtrl', IndexCtrl)
}

export const registerListingCtrl = (module) => {
  module.controller('ListingCtrl', ListingCtrl)
}

export const registerAboutCtrl = (module) => {
  module.controller('AboutCtrl', AboutCtrl)
}

export const registerControllers = (module) => {
  registerIndexCtrl(module)
  registerListingCtrl(module)
  registerAboutCtrl(module)
}

```

NOTE : We now have functions to register each controller separately. They will be really useful in the next step.

and we only import `registerControllers` from `my-module.js` :

```js
// my-module.js

import {
  registerControllers
} from './controllers.js'

var myModule = angular.module('myModule', [])

const myConfig = ($stateProvider) => {

  $stateProvider
  .state('index', {
    url: '/',
    template: require('path/to/my/template.html'),
    controller: 'IndexCtrl as vm',
  })
  .state('listing', {
    url: '/listing',
    template: require('path/to/my/template.html'),
    controller: 'ListingCtrl as vm',
  })
  .state('about', {
    url: '/',
    template: require('path/to/my/template.html'),
    controller: 'AboutCtrl as vm',
  })

}

myModule.config(myConfig)
registerControllers(myModule)
// Still no code splitting here! We didn't change anything in the process!

export default myModule

```

Now that we a have functions that let us register our controllers when we want to, it's time to use Webpack!

#### Code splitting for real

##### Lazy loading templates

As I said in the introduction, code splitting in webpack is triggered using `require.ensure`.

Angular-UI-Router's `$stateProvider` has a really nice way to provide templates programmatically using the [`templateProvider`](http://angular-ui.github.io/ui-router/site/#/api/ui.router.state.$stateProvider) option instead of `template` and `templateURl`



```js
{
  templateProvider: ['$q', function ($q) {
      return new $q((resolve, reject) => {
        require.ensure([], () => { // Split point
            let template = require('path/to/my/template.html');
            resolve(template);
        });
      })
  }]
}
```

Let's define a `onDemandTemplate` function that abstracts the code splitting process for AngularJS templates :

```js
// utils.js

export const onDemandTemplate = (path) => {
  return ['$q', function ($q) {
      return new $q((resolve, reject) => {
        require.ensure([], () => { // Split point
            let template = require(path);
            resolve(template);
        });
      })
  }]
}
```

```js
// my-module.js

import {
  registerControllers
} from './controllers.js'

import {
  onDemandTemplate
} from './utils.js'

var myModule = angular.module('myModule', [])

const myConfig = ($stateProvider) => {

  $stateProvider
  .state('index', {
    url: '/',
    templateProvider: onDemandTemplate('path/to/my/template.html'),
    controller: 'IndexCtrl as vm',
  })
  .state('listing', {
    url: '/listing',
    templateProvider: onDemandTemplate('path/to/my/template.html'),
    controller: 'ListingCtrl as vm',
  })
  .state('about', {
    url: '/',
    templateProvider: onDemandTemplate('path/to/my/template.html'),
    controller: 'AboutCtrl as vm',
  })

}

myModule.config(myConfig)
registerControllers(myModule)

export default myModule

```

If we build and run our app in Chrome now, we should see the fetching of small js bundle the first time we access to a route.

SCREENSHOT HERE


##### Lazy loading controllers


```js
// utils.js

export const onDemandTemplate = (path) => {
  return ['$q', ($q) => {
      return new $q((resolve, reject) => {
        require.ensure([], () => { // Split point
            let template = require(path);
            resolve(template);
        });
      })
  }]
}

export const onDemandController = (path, ngModule) => {
  return ['$q', ($q) => {
    return new $q((res, rej) => {
      require.ensure([], () => { // Split point
        let registerer = require(path)
        registerer.default(ngModule)
        res(module);
      });
    })

  }]
}
```


```js
// my-module.js

import {
  registerControllers
} from './controllers.js'

import {
  onDemandTemplate,
  onDemandController
} from './utils.js'

var myModule = angular.module('myModule', [])

const myConfig = ($stateProvider) => {

  $stateProvider
  .state('index', {
    url: '/',
    templateProvider: onDemandTemplate('path/to/my/template.html'),
    controller: 'IndexCtrl as vm',
    resolve: {
      'whateverTheName': onDemandController()
    }
  })
  .state('listing', {
    url: '/listing',
    templateProvider: onDemandTemplate('path/to/my/template.html'),
    controller: 'ListingCtrl as vm',
  })
  .state('about', {
    url: '/',
    templateProvider: onDemandTemplate('path/to/my/template.html'),
    controller: 'AboutCtrl as vm',
  })

}

myModule.config(myConfig)
// No need to use registerControllers now!

export default myModule

```
