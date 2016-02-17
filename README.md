# dependency-injection

**Dependency Injection in Node.js**

*Dependency injection is a software design pattern in which one or more dependencies (or services) are injected, or passed by reference, into a dependent object.*

***Reasons for using Dependency Injection***

*Decoupling*

Dependency injection makes your modules less coupled resulting in a more maintainable codebase.

*Easier unit testing*

Instead of using hardcoded dependencies you can pass them into the module you would like to use. With this pattern in most cases, you don't have to use modules like proxyquire.

*Faster development*

With dependency injection, after the interfaces are defined it is easy to work without any merge conflicts.

How to use Dependency Injection using Node.js

**Dependency Injection in Real Projects**

You can find dependency injection examples in lots of open-source projects. For example, most of the Express/Koa middlewares that you use in your everyday work uses the very same approach.

*Express middlewares*

```javascript
var express = require('express');  
var app = express();  
var session = require('express-session');

app.use(session({  
  store: require('connect-session-knex')()
}));
```

The code snippet above is using dependency injection with the factory pattern: to the session middleware we are passing the `connect-session-knex` module - it has to implement an interface, that the `session` module will call.

In this case the `connect-session-knex` module has to implement the following methods:

- `store.destroy(sid, callback)`

- `store.get(sid, callback)`

- `store.set(sid, session, callback)`

*That is a text copied from a book, now from my own knowledge:

**At the beginning I did myself to questions**

- *Everytime a require is call in Nodejs, is that named dependency injection? Or what is the real meaning of dependency injection?*

- *The reason why I am asking myself this, is because I've been reading about Node, and I see people people talking about the module or module.export pattern, and I am confuse, the module is the same as dependency?*

***TL; DR;***

Dependency injection is somewhat the opposite of normal module design.  In normal module design, a module uses `require()` to load in all the other modules that it needs with the goal of making it simple for the caller to use your module.  The caller can just `require()` in your module and your module will load all the other things it needs.

With dependency injection, rather than the module loading the things it needs, the caller is required to pass in things (usually objects) that the module needs.  This can make certain types of testing easier and it can make mocking certain things for testing purposes easier.

> Everytime a require is call in Nodejs, is that named dependency
> injection? Or what is the real meaning of dependency injection?

No.  When a module does a `require()` to load it's own dependencies that is not dependency injection.

> The reason why I am asking this, is because I've been reading about
> Node, and I see people people talking about the module or
> module.export pattern, and I am confuse, the module is the same as
> dependency?

A module is not the same as a dependency.  Normal module design allows you to `require()` just the module and get back a series of exports that you can use.  The module itself handles the loading of its own dependencies (usually using `require()` internal to the module).

Here are a couple of articles that discuss some of the pros/cons of using dependency injection.  As best I can tell the main advantage is to simplify unit testing by allowing dependent objects (like databases) to be more easily mocked.

http://stackoverflow.com/questions/1580641/when-to-use-dependency-injection

[When is it not appropriate to use dependency injection][1]

[Why should we use dependency injection][2]

-------------

A classic case for using a dependency injection is when a module depends upon a database interface.  If the module loads it's own database, then that module is essentially hard-wired to that particular database.  There is no architecture built into the module for the caller to specify what type of storage should be used.

If, however, the module is set up so that when a caller loads and initializes the module, it must pass in an object that implements a specific database API, then the caller is free to decide what type of database should be used.  Any database that meets the contract of the API can be used.  But, the burden is on the caller to pick and load a specific database.  There can also be hybrid circumstances where a module has a built-in database that will be used by default, but the caller can supply their own object that will be used instead if it is provided in the module constructor or module initialization.

  [1]: http://programmers.stackexchange.com/questions/135971/when-is-it-not-appropriate-to-use-the-dependency-injection-pattern
  [2]: http://www.javacreed.com/why-should-we-use-dependency-injection/
  
-------------
  
Imagine this code.

```javascript
var someModule = require('pathToSomeModule');

someModule();
```

Here, we DEPEND not on the name, but on the path of that file. We are also using the SAME file every time.

Let's look at angular's way (for the client, i know, bear with me)

```javascript  
app.controller('someCtrl', function($scope) {
  $scope.foo = 'bar';
});
```

I know client side js doesn't have file imports / exports, but it's the underlying concept that you should look at. Nowhere does this controller specify WHAT the $scope variable ACTUALLY is, it just knows that angular is giving it something CALLED $scope.

This is *Inversion of Control*
It is like saying, *Don't call me, I'll call you*

Now let's implement our original code with something like a service container (there are many different solutions to this, containers are not the only option)

```javascript
// The only require statement
var container = require('pathToContainer')

var someModule = container.resolve('someModule');
    
someModule();
```

What did we accomplish here? Now, we only have to know ONE thing, the container (or whatever abstraction you choose). We have no idea what `someModule` actually is, or where it's source file is, just that it's what we got from the container. The benefit of this is that, if we want to use a different implementation of `someModule`, as long as it conforms to the same API as the orignal, we can just replace it ONE place in our ENTIRE app. The container. Now every module that calls `someModule` will get the new implementation. The idea is that when you make a module, you define the api that you use to interact with it. If different implementations all conform to a single api (or you write an adapter that conforms to it) then you can swap out implementations like dirty underwear and your app will just WORK.

This approach is not for everyone, and some people hate lugging around the container.

My personal opinion, I would rather code to an interface ( a consistent api between implentations ) than code to a concrete implementation.


An actual example of Dependency *Injection* in node.js

```javascript
// In a file, far, far, away
module.exports = function(dependencyA, dependencyB) {
  dependencyA();
  dependencyB();
}

// In another file, the `caller`
// This is where the actual, concrete implementation is stored
var depA = someConcreteImplementation;
var depB = someOtherConcreteImplementation;

var someModule = require('pathToSomeModule');

someModule(depA, depB);
```

The downside to this, is now the caller needs to know what your dependencies are. Some are comforted by this and like it, others believe it's a hassle.

I prefer this next approach, personally.

If you aren't using babel, or something that changes your functions behind the scenes, you can use this approach to get the angular-style parameter-parsing

http://krasimirtsonev.com/blog/article/Dependency-injection-in-JavaScript

Then you can parse the function you get from `require`, and not use a container at all.

-------------

Here is a resource where you can read about this

http://www.schibsted.pl/2015/12/how-to-do-dependency-injection-in-node-js/
