# Yii2-angular
####"But wait," you say, "Yii is too magical!"
[view slide deck](http://vybeauregard.github.io/Yii2-angular/present.html#/8)

You're right. Sort of. The magic is in everything Yii does under the covers. And while all of the forms and pages that Gii generates are handy, sometimes they won't _quite_ fit your needs. Especially when you're mixing and mashing up different models in different ways.

But still, the rigid MVC architecture is nice to have and makes reading from and writing to the database ~~DROP TABLE~~ drop-dead easy. This part of it is worth hanging on to.

So how do we roll our own interface *and* integrate Yii's fancy MVC architecture?

I've been using a two-pronged approach and it's proven to be quite flexible:

##Yii's REST service with Angular.js
>[Two great tastes that taste great together!](http://youtu.be/DJLDF6qZUX0#t=5)

####"Hang on there, M&aelig;stro. Isn't Angular overcomplicating things?"

Not any more than using Bootstrap or jQuery overcomplicates things. I'll illustrate how they work together as we move along.

With Yii handling all of your users' requests, every change in data is accompanied with a full page reload. Even with browser caching, that can really slow down the user experience and trigger a ton of redundant server requests. If can we hand off some of the data manipulation to an AJAX-like interface that updates the UI without a full page refresh, we'll end up with a snappier interface and lower network overhead.

Yii is tightly integrated to your database and its models are perfect for providing a RESTful API that returns JSON objects. If you're just learning about [REST](https://en.wikipedia.org/wiki/Representational_state_transfer), Wikipedia is a good place to start.

To put it briefly, REST allows us to send out small requests and receive a response with each one. When implemented, you can use the actions `GET`, `PUT`, `POST`, and `DELETE` to act upon the data store.

How does this help us as we build web apps? Because Angular has these actions built in using its `$http` service, and sends them as [XHR](https://en.wikipedia.org/wiki/XMLHttpRequest). As soon as it receives a response, it updates every single place it's been referenced in the interface with no extra jQuery tricks and tomfoolery, and better still, with just a few KB changing hands.

####Why do you keep bashing jQuery?

I've got nothing but love for jQuery, but like any tool, it's not always the best solution for the challenge we're presented with. Yes, it *can* perform XHR, but it does so with a much lower-level approach than Angular and leaves a lot of the mess of wiring everything together to the poor developer.

####I'm still skeptical, but show me something in action already!

I was about to. Increase your calm.

We're going to continue working with the Cupcakes app from [my previous tutorial](http://vybeauregard.github.io/Yii2-Cupcakes/). If you've been following along, you can pick right up where we left off. If not, clone [this repo](https://github.com/vybeauregard/Yii2-Cupcakes.git), checkout `working-branch`, then run `composer update` and `./yii migrate` to bring yourself up to speed.

Before we can continue, we need to make sure Angular is a part of the assets that composer is pulling in.

[Angular installation walkthrough](https://github.com/vybeauregard/Yii2-angular/wiki/Install-Angular-into-a-Yii2-project)

We'll use Yii's routing to access our Angular page, and we want it to be accessible via `http://localhost/cupcakes`. Let's add an action to our `CupcakesController` now.

`controllers/CupcakesController.php`

```php
. . .
public function actionIndex()
{
  return $this->render('index');
}
. . .
```
This will ensure that our `index.php` page will load when we request `http://localhost/cupcakes`. We'll create that page in a second. 

Next, we need a way to expose our cupcakes data in a format Angular can interpret. Yii happens to have a JSON formatter that works well in just such a situation.

`controllers/CupcakesApiController.php`

```php
<?php
namespace app\controllers;

use Yii;
use app\models\Cupcakes;
use app\controllers\CupcakesController;

class CupcakesApiController extends CupcakesController
{
    public function actionView($id)
    {
        \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
        $cupcakes = new Cupcakes();
        return $cupcakes->viewCupcakeDetails($id);
    }

    public function actionIndex()
    {
        \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
        $cupcakes = new Cupcakes();
        return $cupcakes->viewAllCupcakes();
    }
    
    public function actionUpdate()
    {
        \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
        $request = Json::decode(Yii::$app->request->getRawBody());
        foreach($request as $cupcake){
            $newCupcake = $this->findModel($cupcake['id']);
            $newCupcake->setAttributes($cupcake);
            $newCupcake->save();
        }
    }
    
    protected function findModel($id)
    {
      if (($model = Cupcakes::findOne($id)) !== null) {
        return $model;
      } else {
        return new Cupcake;
      }
  }
}
```
Now let's create `index.php` so our Yii controller can render the view when we ask it to.

`views/cupcakes/index.php`

```html
<html>
  <body ng-app="cupcakes" ng-controller="CupcakeController">
    . . .
  </body>
</html>
```
####Hey! `ng-app` ain't no attribute I ever heard of. What kind of witch doctor are you?

Here's where Angular gets invoked. Any attribute in an html page that starts with `ng-` is called a directive. As the browser renders the page, Angular takes any `ng-` attribute and evaluates it as necessary. This keeps your html looking like html while allowing for dynamic data manipulation at the same time.

You can even create your own app-specific directives:

`views/cupcakes/index.php`

```html
<html>
  <body ng-app="CupcakeApp" ng-controller="cupcakesController">
    <table class="table table-bordered table-collapse table-striped">
      <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Description</th>
            <th>Cake Flavor 1</th>
            <th>Cake Flavor 2</th>
            <th>Cake Color</th>
            <th>Icing Flavor</th>
            <th>Icing Color</th>
            <th>Fondant?</th>
            <th>Calories</th>
        </tr>
      </thead>
      <tbody>
        <tr ng-repeat="cupcake in cupcakesList">
          <td class="cupcake-id"></td>
          <td class="cupcake-name"></td>
          <td class="cupcake-description"></td>
          <td class="cupcake-cake-flavor-1"></td>
          <td class="cupcake-cake-flavor-2"></td>
          <td class="cupcake-cake-color"></td>
          <td class="cupcake-icing-flavor"></td>
          <td class="cupcake-icing-color"></td>
          <td class="cupcake-fondant"></td>
          <td class="cupcake-calories"></td>
        </tr>
      </tbody>
    </table>
  </body>
</html>
```
In this example, we're using **class** directives. You can also create directives on **attributes** (e.g. `<div cupcake-description></div>`) and **elements** (e.g. `<cupcake-description></cupcake-description>`). Or, if you just need a quick element of your object's model injected into the page, you can just drop it in: `{{cupcake.description}}`

Let's create these directives in our app and really Angularify this page!

In `application.js`, we will instantiate the module and inject its dependencies in the brackets.

`web/js/application.js`
```js
angular.module('CupcakeApp', [
  'CupcakeApp.controllers',
  'CupcakeApp.services',
  'CupcakeApp.directives'
]);
```
Now, we will create the dependencies:
`cupcakesController` tells the app how to obtain a cupcake object:

`web/js/controllers.js`
```js
angular.module('CupcakeApp.controllers', [])
    .controller('cupcakesController', function($scope, cupcakeAPIservice) {
    $scope.cupcakesList = [];

    cupcakeAPIservice.getCupcakes().success(function (response) {
        $scope.cupcakesList = response;
    });
    
    $scope.saveCupcakes = function(){
      cupcakeAPIservice.saveCupcakes($scope.cupcakesList).success(function (response) {
        $scope.cupcakesList = response;
      });
    }
});
```
Next, our `cupcakeAPIservice` will tell the app where the cupcake data is stored. In this case, it's exposed by our `cupcakes-api`.

`web/js/services.js`
```js
angular.module('CupcakeApp.services', [])
    .factory('cupcakeAPIservice', function($http) {

    var cupcakeAPI = {};

    cupcakeAPI.getCupcakes = function() {
        return $http({
            method: 'GET',
            url: 'cupcakes-api/index'
        });
    };
    cupcakeAPI.saveCupcakes = function(data) {
        return $http({
            method: 'PUT',
            data: data,
            url: 'cupcakes-api/update'
        });
    };

    return cupcakeAPI;
});
```

Since Yii already has a `cupcake` model defined, the JSON object it returns will pass everything along to Angular without the need to redefine the same object there too.

The last dependency we have to create contains our `directives`. Here is where Angular takes our custom class names and replaces its contents with attributes from our cupcake model using templates we define:

`web/js/directives.js`
```js
angular.module('CupcakeApp.directives', [])
    .directive('cupcakeId', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.id}}</td>'
        }
    }).directive('cupcakeName', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.name}}</td>'
        }
    }).directive('cupcakeDescription', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.description}}</td>'
        }
    }).directive('cupcakeCakeFlavor1', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.cake_flavor_1}}</td>'
        }
    }).directive('cupcakeCakeFlavor2', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.cake_flavor_2}}</td>'
        }
    }).directive('cupcakeCakeColor', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.cake_color}}</td>'
        }
    }).directive('cupcakeIcingFlavor', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.icing_flavor}}</td>'
        }
    }).directive('cupcakeIcingColor', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.icing_color}}</td>'
        }
    }).directive('cupcakeFondant', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.fondant}}</td>'
        }
    }).directive('cupcakeCalories', function() {
        return {
            restrict: 'CE',
            replace: 'true',
            template: '<td>{{cupcake.calories}}</td>'
        }
    });
```
You'll notice in the template definition of each of these directives we use double curly brace notation. This tells angular we want to access a property of our `cupcake` object, as defined by our controller.

Now that we have all of these extra JS files, we need to tell Yii to load them when the app is requested.

`assets/AppAsset.php`
```php
. . .
public $js = [
  'js/application.js',
  'js/controllers.js',
  'js/directives.js',
  'js/services.js',
];
. . .
```
Now refresh your app and you should see Angular work its magic.

####That was an awful lot of work for a static table. Yii already gave us that out of the box.

You're right. If we're just spitting out data and not exposing any updating capability to the user, this isn't a good solution. But let's say our cupcake baker wants to batch update her entire inventory of cupcakes without a bunch of back and forth jiggery-pokery?

First, we'll need to update the cupcakes view by replacing our class directive table fields with form fields. This time we will be using Angular's built-in `ng-model` directive instead of our custom directives. `ng-model` performs **two-way data binding**. That means that any changes the user makes are immediately refleced in Angular's model of that object in **real time**.

`views/cupcakes/index.php`

```html
<body ng-app="CupcakeApp" ng-controller="cupcakesController">
  <table class="table table-bordered table-collapse table-striped">
    <thead>
      <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Description</th>
          <th>Cake Flavor 1</th>
          <th>Cake Flavor 2</th>
          <th>Cake Color</th>
          <th>Icing Flavor</th>
          <th>Icing Color</th>
          <th>Fondant?</th>
          <th>Calories</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="cupcake in cupcakesList">
        <td>{{cupcake.id}}</td>
        <td><input class="form-control" ng-model="cupcake.name" /></td>
        <td><textarea class="form-control" ng-model="cupcake.description"></textarea></td>
        <td><input class="form-control" ng-model="cupcake.cake_flavor_1"></td>
        <td><input class="form-control" ng-model="cupcake.cake_flavor_2"></td>
        <td><input class="form-control" ng-model="cupcake.cake_color"></td>
        <td><input class="form-control" ng-model="cupcake.icing_flavor"></td>
        <td><input class="form-control" ng-model="cupcake.icing_color"></td>
        <td><input class="form-control" ng-model="cupcake.fondant"></td>
        <td><input type="number" class="form-control" ng-model="cupcake.calories"></td>
      </tr>
    </tbody>
  </table>
  <input class="btn btn-success" type="button" value="Hello" ng-click="saveCupcakes()" />
</body>
```

Go ahead and comment out `CupcakeApp.directives` in `application.js`. After this change, it is no longer required.

`web/js/application.js`

```js
angular.module('CupcakeApp', [
  'CupcakeApp.controllers',
  'CupcakeApp.services',
  // 'CupcakeApp.directives'
]);
```

Likewise, including `js/directives.js` in `AppAsset.php` is no longer necessary:

`assets/AppAsset.php`

```php
public $js = [
  'js/application.js',
  'js/controllers.js',
  // 'js/directives.js',
  'js/services.js',
];
```

Now you'll see the same exact table, except all the data fields are now editable. We've already got the behavior wired up in our controller and service definitions, so we should now be able to save any changes made to this cupcake data.


