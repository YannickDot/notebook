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

NOTE : We now have functions to register each controller separately. We will use these in the next step.

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

Now that we a have a function that let us register our controllers when we want to, it's time to use Webpack!

#### Code splitting for real
