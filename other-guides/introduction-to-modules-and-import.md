# Introduction to Modules in Node.js

While some of the things about modules have been covered in the [Command Handler](/coding-guides/a-basic-command-handler.md) page, I feel that it would be very constructive to introduce the idea of "modules" separately, unrelated to how they are used in discord.js bots. So if you're on this page and wonder "what the hell is discord.js?" don't worry, this guide is still for you! 

In this guide we'll be taking a look specifically at how to create a module for node.js starting from extremely basic one-line all the way to explaining complex implementations with multiple functionality. But don't worry, we'll keep it as simple as possible!

> In this guide as all my others, I use the latest JavaScript syntax available, which can variably called ES6, ES2017, or "Node" JavaScript. For instance, I use `const functionName = (arg) => {}` to create functions, which is the "fat arrow" way of doing things. I apologize if you're not used to this, check out [this article for more info](https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/)!

## What's a module?

A "module" in node.js is a separate file \(or group of files\) containing "things" that we can use in another file. Modules can contain functions, variables, classes, basically anything that node.js code can contain. So you might ask, why use a module at all if I can just write all my code in a single file? It's about two things: readability, and portability. 

First let's address **readability**: a complex program, even written in node.js, can be thousands of lines of code. Having 6000 lines in a single file is not only difficult to read, it's also difficult to figure out what does what. Furthermore, there are technical problems with a single file code: some parts of the code can affect others, like overwriting variables on a higher scope, changing methods, etc. It's what is called "side-effects". On a separate module, the module can only really affect itself unless you give it something to change.

The second issue is **portability**. Programmers don't generally re-write everything from the ground up every time we start up a new project, right? We re-use parts of our code throughout various applications, and sometimes even update that code in multiple applications at once when we find an issue with it. By creating a module, this makes it easier: if you have a module file you plop in and require in multiple projects, just overwrite that file and boom, you're done!

There's also the idea of **distributing **modules. However, this is beyond the scope of this guide. If you're interested, check out [Publishing NPM Packages on npmjs.org](https://docs.npmjs.com/getting-started/publishing-npm-packages).

## Our First Module

Ok so, let's start with the super basic, and yet not very useful, module that returns a static string. For a moment imagine you have a folder called `myTest` and in this folder, you have 2 files: `index.js` and `myModule.js`.

```js
// This is index.js
console.log("Start Testing");

const myMessage = require("./myModule.js");

console.log(myMessage);
```

```js
// This is myModule.js
module.exports = "Hello, World!";
```

Running `node index.js` in this folder would produce the following console output: 

```
> node index.js
Start Testing
Hello, World!
> _
```

Let's explain exacly why, and how a module "replaces" regular code. You could write the exact same code in a single file as such: 

```js
console.log("Start Testing");
const myMessage = "Hello, World!";
console.log(myMessage);
```

The difference with the module, is simply that it's placed in a separate file. Ok so, like. Yeah, that's a module. It's as simple as that! But of course, there's more to learn here, so let's push on to the next step. 

## A function in a module

Obviously, having a module return a static string isn't super useful. So, most likely we want to have some sort of functionality attached to this module. Let's continue on the simple path, and make a module that creates a "welcome message". I'm sure you remember doing this stuff while learning javascript, right? Here's a function you might have seen, or even written, before:

```js
const hello = (name) => {
 return "Hello, " + name + ", to this world!";
}

const message = hello("John");
console.log(message); // "Hello, John, to this world!"
```

Let's take this simple hello module containing that function instead, just to see how that works out.

```js
// helloWorld.js
module.exports = (name) => {
 return "Hello, " + name + ", to this world!";
}
```

```js
// index.js
const hello = require("./helloWorld.js");
const message = hello("John");
console.log(message); // "Hello, John, to this world!"
```

As you can see, there really isn't that much to it. Creating a module with a single function or a single return is that simple. Let's continue exploring a bit more, though!

## Module Scoping

Ok so, one thing that's really cool about modules is that you can use other modules into them. I'm going to go _a little too far_ into code complexity right now by creating a simple module that gets a random cat image from an online API that's literally called "Random Cat". Bear with me for a second: 

```js
// randomcat.js
const snekfetch = require("snekfetch");

module.exports = async () => {
  const response = await snekfetch.get("http://random.cat/meow");
  return response.body.file;
}
```

```js
// index.js
const randomCat = require("./randomcat.js");
console.log( randomCat() );
```

Ok so, what am I doing here? First off, I'm requiring `snekfetch`, [a simple, fast library that does HTTP queries](https://www.npmjs.com/package/snekfetch). Then I'm using this library inside my module to go "fetch" a single image file from the random cat API and return it. Why is this useful? Well simply put, it means that if this is the only file where I'm using snekfetch, it's the only place where I call it. If we were dealing with a complex application, it would be hard to see where each module is used in the code because you'd have all those require\(\) lines at the top of one file. In separate modules, you know exactly what module uses what library.

Another thing you might notice is that the line that requires snekfetch is outside of the actual module exports, but... why? Well, here's the thing: When you require a module, the whole file is "parsed" once \(in other words, it's executed\). Anything you define in that file is still defined as usual _except that it's only available in that module_. So, snekfetch is only available from the `randomcat.js` file and you can't call it from outside. As far as `randomcat.js` is concerned, it's providing a single function that returns a file, and that's it. 

So if you define strings, objects, arrays, or whatever else outside of your module and they'll be "private" to that module! It makes it a self-contained entity that can only interact and be interacted with in specific manners. 

## Multiple Returns

Now that we've seen modules in their simplest forms, we're ready to thrown in another requirement into the mix: We want to be able to add more than one thing to this module. Say we have a "utility" module that has a couple functions we like to use in our applications. Let's make _that_!

```js
// utils.js
module.exports = "A utility Module by Evie";

module.exports.toProperCase = (myString) => {
  return myString.replace(/([^\W_]+[^\s-]*) */g, (txt) => {
   return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();
  });
}

module.exports.arrayRandom = (myArray) => {
  return myArray[Math.floor(Math.random() * myArray.length)];
}

module.exports.helloWorld = (name) => {
  return "Hello, " + name + ", to this world!";
}
```

So now we have a `utils.js` module with 3 different useful functions that we use all over the place every day. How do we call it? Almost the same as we did before, except we can use its named methods!

```js
// index.js

const utils = require("./utils.js");

console.log(utils.toProperCase("this is a sentence")) // This Is A Sentence
console.log(utils.arrayRandom([1, 2, 3, 4])) // a random number from 1 to 4
console.log(utils.helloWorld("Evie")) // Hello, Evie, to this world!
```

## More Scopes

Let's return for a moment to "scoping" in modules. One thing that you might want to consider doing with modules is to have "properties" that can be changed by the module and accessed externally. What do I mean by that? Let's make a super simple ToDo module to demonstrate. Instead of blabbering on in paragraphs after, I'll use comments in the code to convey what's happening. 

```js
// todo.js

// surprise, `module.` is optional!
exports.todoList = [];
// Accessible through `this.todoList` because `this` is the module itself!

// Method to add a new thing to the list
exports.add = (thingtodo) => {
  // this.todoList is accessing the above array.
  // If the items isn't present (remove duplicates), add it:
  if(this.todoList.indexOf(thingtodo) < 0)
    this.todoList.push(thingtodo);
  // We return our whole module for "chaining" actions. See usage, below.
  return this;
}

// Same deal but we remove the list from its index position.
exports.remove = (thingtodo) => {
  const pos = this.todoList.indexOf(thingtodo);
  this.todoList = this.todoList.slice(pos, 1);
  return this;
}

// A Simple clear function that removes all elements.
exports.clear = () {
  this.todoList = [];
  return this;
}

// This function "gets" all the ToDos: 
exports.list = () => {
  // because we return only an array, it can't be chained after.
  return this.todoList;
}

// A "Clean" return of all the ToDos with line returns and numbers!
exports.cleanList = () => {
  // oh man this is awesome ES6 and I wish you understand it!
  // Things used: Array.prototype.map , Array.join, and Template Literals
  return this.todoList.map( (item, index)  => `${index}. ${item}`).join("\n");
}
```

With this we have a module with a **public property** called `todoList` that can actually be modified externally, as well as 4 **public methods** to modify that list a little easily. Let's see how to use it. 

```js
// index.js
const groceries = require("./todo.js");

console.log(groceries.cleanList()) // outputs an empty string

// Add Milk
groceries.add("Milk");

// Add Bread, add butter, remove bread
groceries.add("Bread").add("Butter").remove("Bread");

console.log(groceries.cleanList());
/* 
1. Milk
2. Butter
(notice the absense of Bread since it was removed)
*/

// I can also add things manually if I want to but it can be bad!
groceries.todoList.push("Butter");

// Duplicate because I didn't check for that, `add()` does.
console.log(groceries.list()); // ["Milk", "Butter", "Butter"]

groceries.clear(); // it's now empty!
```

If you only wanted this kind of module, you are now done with this guide. You can skip the final parts if you're not interested in classes, instances, or different syntaxes. You have enough to be functional! 

## Alternate Syntax

Let's just take a quick look at a different way to make a module, using a different syntax. This can be used to make, let's say, "Private" properties and methods, to avoid the issue above with adding to the ToDo list from outside. I feel like this is relatively self-explanatory at this point. If it's not, either you haven't been following well or I need to review my teaching methodology!

```js
// todo.js

todoList = [];

const add = (thingtodo) => {
  if(todoList.indexOf(thingtodo) < 0)
    todoList.push(thingtodo);
  return this;
}

const remove = (thingtodo) => {
  const pos = todoList.indexOf(thingtodo);
  todoList = todoList.slice(pos, 1);
  return this;
}

const clear = () => {
  todoList = [];
  return this;
}

// I've simplified this one
const list = () => todoList;

// And this one too, taking advantage of code simplification in ES6.
const cleanList = () => this.list().map((o, i)=>`${i}. ${o}`).join("\n");

// Now we EXPORT only thing things we need.
module.exports.add = add;
module.exports.remove = remove;
module.exports.list = cleanList; // so we can use this.list() internally only!
// notice the lack of exporting todoList!
```

## Modules as Classes

One last thing before we go, classes. An advantage of adding a class to a module is that it can be re-used so much more easily, even if that class relies on external libraries or specific internal private code and methods. Let's go back to the basics with the example of a class ripped straight off MDN. 

```js
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}
```

You can stick this in any file and you can create a new rectangle using `const myRect = new Rectangle(10, 5)`, sure. But then you might be worries about re-usability and conflicts with other code, right? So let's make it a module using what we've learned before. 

```js
// rectangle.js
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

module.exports = Rectangle;
```

```js
//index.js
const Rectangle = require("./rectangle.js");

const myRect new Rectangle(10, 5); // works the same!
```

With this idea we can now require modules and libraries only from a class file and make this code so much prettier. 

You can also easily extend classes this way, just as easily. 

```js
const Rectangle = require("./rectangle.js");

class ColoredRectangle extends Rectangle {
  constructor(height, width, color) {
    super(height, width);
    this.color = color;
  }
}

module.exports = ColoredRectangle;
```

## Notes and Addendums

### Path Relativity

`require()` uses a path **relative to your project root**. This means, in all the above examples we assume that you are in the root folder and all the files are there. If you're in `./index.js` and you call `./rectangle.js` it's from the root folder. If you had a folder called `shapes` and you put the classes in it, you could call `require("./shapes/rectangle.js")`. However, what's less known is that if you had a file in another subfolder, say, `src`, from `./src/myapp.js` you would **still** call it using `./shapes/rectangle.js` because that's where it is relative to the root folder. **This is contrary to the fs module** in which the path is relative to the current file.

You can also require files from other projects by going up folders. So if I have `project1` and `project2` in my development folder, from `Project2` I can easily do `require("../project1/somefile.js");` . While this is useful in development, it is also very critical to remember if ever you make anything where you require things dynamically.

### Publishing Modules/Libraries

I've touched upon publishing in my introduction. Publishing modules is done on NPM \(though the node community wants to enable requires from an HTTP page, that's not doable yet\), and anyone can publish a module, if the name is unique. To "install" a published library you just need to use npm. For example, in the random cat example I use snekfetch, which needs to be installed using `npm install snekfetch`. An installed library doesn't use a "path" or a .js extension so it's `require("snekfetch")` instead of, say, `require("./snekfetch.js")`.

