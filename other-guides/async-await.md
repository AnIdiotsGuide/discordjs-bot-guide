---
description: >-
  Learn how to use the awesome node feature called async/await to simplify your
  code that uses promises.
---

# Async / Await

When an async function is called, it returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). When the async function returns a value, the Promise will be resolved with the returned value. When the async function throws an exception or some value, the Promise will be rejected with the thrown value.

An async function can contain an [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) expression, that pauses the execution of the async function and waits for the passed Promise's resolution, and then resumes the async function's execution and returns the resolved value.

### Promise

But, what are Promises? Promise Object is used for asynchronous computations/calls, but unlike functions, they don't return the value immediately, as they have three states:

* **Pending**: initial state, not fulfilled or rejected.
* **Fulfilled**: meaning that the operation completed successfully.
* **Rejected**: meaning that the operation failed.

A function returns a Promise when you call an asynchronous method, it means, as explained above, the **final** value isn't available when the function has been called, but when the object Promise resolves.

An example in the real world would be:

**You get a bottle of water, you open it, turn over and drain it. Then dispose it.**

The problem above is, when you get the bottle of water, you can immediately open it \(sync function\), but when you turn it over and drain it, you have to **await** until the bottle gets empty. This is, an **AsyncFunction**.

**AoDude\#8676** proposed the following example:

```javascript
const perrier = require("PerrierBrandWater");
const bottle = new perrier.BottleOfWater();

bottle.open(); // sync operation
bottle.turnOverAndDrain() // async operation
    .then(emptyBottle => emptyBottle.dispose())
    .catch((err) => {
        console.error(err);
        runForYourLives();
    });

const runForYourLives = () => process.exit();
```

In the example above, you open the bottle \(sync operation, you can do that in the code execution\), then you call `turnOverAndDrain`, in which is an async operation. You don't know if the operation will be executed successfully \(the water gets completely drained\) or it'll fail \(something happened and the water couldn't get completely drained\).

When you call an asynchronous function, in [**ES6**](https://www.ecma-international.org/ecma-262/6.0/), you can use the keywords `then` and `catch`. At some point they make some logic.

You open the bottle, turn it over and drain, **then** you dispose it. But if something failed, there's an `error`, you **catch** it, display a console error and run for your lives.

Inside `then` and `catch` there are functions, inside then, we use the variable **emptyBottle**, in which is the value that the function **turnOverAndDrain** returns when it resolves. And inside catch, **err** is the error object that the function returns when it fails.

Don't get it? There's an example with Discord.JS:

Some practise, the method [client.fetchUser](https://discord.js.org/#/docs/main/master/class/client?scrollTo=fetchUser) returns `Promise<User>`. It means, when you call that method, it'll return a **Promise**, resolving with a **User** object \(but it can also throw an error\).

```javascript
client.fetchUser(id)
    .then((User) => {
        // Do something with the User object
    })
    .catch((err) => {
        // Do something with the Error object, for example, console.error(err);
    })
```

The code above requires a UserID, but you don't have the User object since you are going to retrieve information from Discord to get the User object, that takes a while, once you get the data, Discord.JS will resolve the method, running the function inside the `then`, passing the object `User`, described in the docs.

### Async/Await usage

Once we know how to use the `Object Promise` and we know how to work with it, it's now time to learn how to use ES8 Promises, with `async`/`await`.

```javascript
async () => {
    const User = await client.fetchUser(id);
    // Do something with the User object
}
```

**WAIT WHAT? THAT'S ALL?** Yes, it is. in the code above, you're defining the constant `User` as the result of the Promise, hence the keyword `await`. In this context, your code \(when it executes\), calls the method `client.fetchUser()`, but it'll stop there, once the promise resolves, the returned value \(User Object\) is assigned to the constant User.

**Wait, we have the replacement for** `then`**, but what if the method fails?** An advantage of ES8 Async/Await is that, you can call multiple AsyncFunctions, and catch them all once. As in the following example:

```javascript
async () => {
    try {
        const User = await client.fetchUser(id);
        const member = await guild.fetchmember(User);
        const role = guild.roles.find("name", "Idiot Subscribers");
        await member.addRole(role);
        await channel.send("Success!");
    } catch (e) {
        console.error(e);
    }
}
```

In the example above, you fetch a user, once you have the User object, declare it as the constant `User`, then you fetch a member with the `User`, if it's found, it'll get a role \(sync method, doesn't return a `Object Promise`, so you don't need the `await` keyword\), add the role to the member, and send a message to the channel.

If **ONE** of the promises fail, the code stops executing the rest of the methods inside the `try`'s block and will run the block inside `catch`, with the Error object, and will send an error to the console.

The example executes a **Promise Chain** \(runs promises, one after the previous\), you don't want to see them with ES6 async/await. Seriously. It's something we call **Callback Hell** \(a lot of indentation levels, code that is very hard to follow, hence much harder to work with...\).

## Important!

To use the `await` keyword, you **MUST** have written the `async` keyword in the function whose block contains the code. For example:

```javascript
const EditMessage = async (id, content) => {
    const Message = await channel.fetchMessage(id); // Async
    return Message.edit(content);
}
```

```javascript
async function EditMessage(id, content) {
    const Message = await channel.fetchMessage(id); // Async
    return Message.edit(content);
}
```

However, if you have a function inside another, for example:

```javascript
const EditMessage = async (id, content) => {
    const Message = await channel.fetchMessage(id);
    setTimeout(() => {
        await Message.edit(content);
        Message.channel.send("Edited!");
    }, 5000);
}
```

That will throw an error:

```javascript
        await Message.edit(content);
              ^^^^^^^
SyntaxError: Unexpected identifier
```

This example failed because the function inside `setTimeout` doesn't have the `async` keyword.

{% hint style="warning" %}
**REQUIREMENTS** You must have Node.js v7.x to use Async/Await, since v7.6 you are not required to use the flag `--harmony` when you run your bot. And Discord.JS v12 will **require** Node.js v8.x. To know your **Node version**, run `node -v` or `node --version`. Download Node.JS v7.x [here](https://nodejs.org/en/download/), but if you want to install it with the package manager \(console commands\), follow the instructions [here](https://nodejs.org/en/download/package-manager/).
{% endhint %}

## Documentation

The following links are from MDN \(Mozilla Developer Network\), they provide syntax, description and examples.

* [JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
* [AsyncFunction \(Statement\)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* [AsyncFunction \(Operator\)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/async_function)
* [await \(Operator\)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
* [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

