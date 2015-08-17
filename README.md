# Yii2-angular
####"But wait," you say, "Yii is too magical!"

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
This will ensure that our `index.php` page will load when we request `http://localhost/cupcakes`. Let's create that page now.

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

Directives can take a couple of other forms too:
```html
<html>
  <body ng-app="cupcakes" ng-controller="CupcakeController">
    <div id="element-directives">
      <cupcake-name></cupcake-name>
      <cupcake-description></cupcake-description>
      <cupcake-cake-color></cupcake-cake-color>
    </div>
    <div id="class-directives">
      <div class="cupcake-name"></div>
      <div class="cupcake-description"></div>
      <div class="cupcake-cake-color"></div>
    </div>
  </body>
</html>
```
**Element** directives allow you to create your own tags outside the HTML spec. Wacky!

**Class** directives allow you to bind data to a container based off its class name. Handy if you need css styles to match the container's contents 100% of the time!

With the appropriate directive setup in your `CupcakeController`, both of its child `div`s above should render the same content.

Let's create a directive and really Angularify this page!
`web/js/application.js`
```js
(function () {
}).load(this);
```
