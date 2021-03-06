#### [<img src="../assets/logo.svg" width="12" height="12" alt="Stimulus">](../README.md) [The Stimulus Handbook](README.md)

---

# 2 Hello, Stimulus

The best way to learn how Stimulus works is to build a simple controller. This chapter will show you how.

## Prerequisites

To follow along, you'll need a running copy of the [`stimulus-starter`](https://github.com/stimulusjs/stimulus-starter) project, which is a preconfigured blank slate for exploring Stimulus.

We recommend [remixing `stimulus-starter` on Glitch](https://glitch.com/edit/#!/remix/stimulus-starter) so you can work entirely in your browser without installing anything:

[<img src="https://cdn.glitch.com/2bdfb3f8-05ef-4035-a06e-2043962a3a13%2Fremix%402x.png?1513093958726" alt="Remix on Glitch" aria-label="remix" height="33">](https://glitch.com/edit/#!/remix/stimulus-starter)

Or, if you'd prefer to work from the comfort of your own text editor, you'll need to clone and set up `stimulus-starter`:

```
$ git clone https://github.com/stimulusjs/stimulus-starter.git
$ cd stimulus-starter
$ yarn install
$ yarn start
```

Then visit http://localhost:9000/ in your browser.

(Note that the `stimulus-starter` project uses the [Yarn package manager](https://yarnpkg.com/) for dependency management, so make sure you have that installed first.)

## It All Starts With HTML

Let's begin with a simple exercise: a text field with a button. When you click the button, we'll display the value of the text field in the console.

Every Stimulus project starts with HTML, and this project is no exception. Open `public/index.html` and add the following markup just after the opening `<body>` tag:

```html
<div>
  <input type="text">
  <button>Greet</button>
</div>
```

Reload the page in your browser and you should see the text field and button.

## Controllers Bring HTML to Life

At its core, Stimulus' purpose is to automatically connect DOM elements to JavaScript objects. Those objects are called _controllers_.

Let's create our first controller by extending the framework's built-in `Controller` class. Create a new file named `hello_controller.js` in the `src/controllers/` folder. Then place the following code inside:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
}
```

## Identifiers Link Controllers With the DOM

Next, we need to tell Stimulus how this controller should be connected to our HTML. We do this by placing an _identifier_ in the `data-controller` attribute on our `<div>`:

```html
<div data-controller="hello">
  <input type="text">
  <button>Greet</button>
</div>
```

Identifiers serve as the link between elements and controllers. In this case, the identifier `hello` tells Stimulus to create an instance of the controller class in `hello_controller.js`.

## Is This Thing On?

Reload the page in your browser and you'll see that nothing has changed. How do we know whether our controller is working or not?

One way is to put a log statement in the `connect()` method, which Stimulus calls each time a controller is connected to the document.

Implement the `connect()` method in `hello_controller.js` as follows:
```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  connect() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

Reload the page again and open the developer console. You should see `Hello, Stimulus!` followed by a representation of our `<div>`.

## Actions Respond to DOM Events

Now let's see how to change the code so our log message appears when we click the "Greet" button instead.

Start by renaming `connect()` to `greet()`:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

We want to call the `greet()` method when the button's `click` event is triggered. In Stimulus, controller methods which handle events are called _action methods_.

To connect our action method to the button's `click` event, open `public/index.html` and add a magic `data-action` attribute to the button:

```html
<div data-controller="hello">
  <input type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

> ### 💡 Action Descriptors Explained
>
> The `data-action` value `click->hello#greet` is called an _action descriptor_. This particular descriptor says:
> * `click` is the event name
> * `hello` is the controller identifier
> * `greet` is the name of the method to invoke

Load the page in your browser and open the developer console. You should see the log message appear when you click the "Greet" button.

## Targets Locate Important Elements By Name

We'll finish the exercise by changing our action to say hello to whatever name we've typed in the text field.

In order to do that, first we need a reference to the input element inside our controller. Then we can read the `value` property to get its contents.

Stimulus lets us mark important elements as _targets_ so we can easily reference them by name. Open `public/index.html` and add a magic `data-target` attribute to the input element:

```html
<div data-controller="hello">
  <input data-target="hello.name" type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

> ### 💡 Target Descriptors Explained
>
> The `data-target` value `hello.name` is called a _target descriptor_. This particular descriptor says:
> * `hello` is the controller identifier
> * `name` is the target name

If we pass the target name to the `this.targets.find()` method, Stimulus will return the first matching target element it finds. We can then read its `value` and use it to build our greeting string.

Let's try it out. Open `hello_controller.js` and update the `greet()` method like so:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    const element = this.targets.find("name")
    const name = element.value
    console.log(`Hello, ${name}!`)
  }
}
```

Then reload the page in your browser and open the developer console. Enter your name in the input field and click the "Greet" button. Hello, world!

## Controllers Simplify Refactoring

We've seen that Stimulus controllers are instances of JavaScript classes whose methods can act as event handlers.

That means we have an arsenal of standard refactoring techniques at our disposal. For example, we can clean up our `greet()` method by extracting a `name` getter:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    console.log(`Hello, ${this.name}!`)
  }

  get name() {
    const target = this.targets.find("name")
    return target.value
  }
}
```

And we can further clean up the `name` getter by extracting an `inputElement` getter for the target element:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    console.log(`Hello, ${this.name}!`)
  }

  get name() {
    return this.inputElement.value
  }

  get inputElement() {
    return this.targets.find("name")
  }
}
```

Now the most important code lives at the top of our controller and its supporting details live below.

## Wrap-Up and Next Steps

Congratulations—you've just written your first Stimulus controller!

We've covered the framework's core concepts: controllers, identifiers, actions, and targets. In the next chapter, we'll see how to put those together to build a real-life controller taken right from Basecamp.

---

Next: [Building Something Real](03_building_something_real.md)
