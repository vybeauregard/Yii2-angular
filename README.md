# Yii2-angular
####"But wait," you say, "Yii is too magical!"

You're right. Sort of. The magic is in everything Yii does under the covers. And sometimes all of the forms and pages that Gii generates won't _quite_ fit your needs, especially when you're mixing and mashing up different models in different ways.

But still, the rigid MVC architecture is nice to have and makes reading from and writing to the database ~~DROP TABLE~~ drop-dead easy. This part of it is worth hanging on to.

So how do we roll our own interface *and* integrate Yii's fancy MVC architecture?

I've been using a two-pronged approach and it's proven to be quite flexible:

##Yii's REST service with Angular.js
####"Hang on there, M&aelig;stro. Isn't Angular overcomplicating things?"

Not any more than using Bootstrap or jQuery overcomplicates things. I'll illustrate how they work together as we move along.

>[Two great tastes that taste great together!](http://youtu.be/DJLDF6qZUX0#t=5)

While we won't need get into which of the two is Chocolate and which is Peanut Butter, here's a quick defense of the Yii/Angular approach:

With Yii handling all of your users' requests, every change in data is accompanied with a full page reload. Even with browser caching, that can really slow down the user experience and trigger a ton of redundant server requests. If can we hand off some of the data manipulation to an AJAX-like interface that updates the UI without a full page refresh, we'll end up with a snappier interface and lower network overhead.

Yii is tightly integrated to your database and its models are perfect for providing a RESTful API that returns JSON objects. If you're just learning about [REST](https://en.wikipedia.org/wiki/Representational_state_transfer), Wikipedia is a good place to start.

To put it briefly, REST allows us to send out small requests and receive a response with each one. When implemented, you can use the actions `GET`, `PUT`, `POST`, and `DELETE` to act upon the data store.

How does this help us as we build web apps? Because Angular has these actions built in using its `$http` service, and sends them as [XHR](https://en.wikipedia.org/wiki/XMLHttpRequest). As soon as it receives a response, it updates every single time it's been referenced in the interface with no extra jQuery tricks and tomfoolery.
