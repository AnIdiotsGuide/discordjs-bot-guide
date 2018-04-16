# Introducing Enmap

Enmap is a data structure that can be used to store data in memory that is also optionally saved in a database behind the scenes. When persistence is enabled, the data is synchronized to the database automatically, seamlessly, and asynchronously.

If persistence is turned off, Enmap acts as a regular Discord.js Collection object, with all the awesome features of Array Methods wrapped into Javascript's native Map\(\) object.

> This how-to will concentrate on using **persistent** Enmap instances. If you don't want persistence, just do `const myCollection = new Enmap();` and use it as you would a native Discord.js Collection!

So why use this when you have so many possible databases to choose from? The goal of Enmap is to make things dead simple for beginner users. You don't have to write any code to read or write from files, no SQL queries, no complications. You just insert data inside your Enmap, and it's saved in the database. That's it!

## Installing

Enmap can be installed from `npm`:

```js
npm i enmap
```

[View the package on NPMJS](https://www.npmjs.com/package/enmap)

You then need to require the module in your code:

```js
const Enmap = require('enmap');
```

You also need a **Provider** for persistence. There are a few available and more upcoming but for now we'll use the basic leveldb on. Fast, efficient, but basic.

```js
const EnmapLevel = require('enmap-level');
```

> `enmap-level` does not support sharding or multiple processes, so it's only useful for small bots. However, there are many other providers you can choose from, linked from [Enmap's Readme](https://www.npmjs.com/package/enmap).

## Creating a "Table"

Most of us understand the concept of a "table", basically a structure that holds multiple rows and columns. This applies to concepts from Databases to Excel Spreadsheets so to simplify this how-to I'll go ahead and use those terms.

To create a new table, then, you need to "initialize" a new Enmap object and the provider:

```js
const tableSource = new EnmapLevel({name: "myTable"});
const myTable = new Enmap({provider: tableSource});
```

When this code is run, one of 2 things can happen:

* **If there is no table of that name**, this table is created in the database, and it's empty.
* **If the table exists**, that table and _all its contents_ is loaded into the Enmap itself.

So what does this mean? It means that "initializing the database" and "loading all its data" is taken care of, for you. You don't need to do anything else for this to happen.

## Inserting Data

Inserting data is super simple once you've initialized the table. Please note however that inserting data has its limits:

* **Keys** must be either a string or a number. Any other value is rejected.
* **Values** must be simple data types on which JSON.stringify\(\) can be applied. Arrays, Objects, Strings, Numbers, those are fine. However, complex types like Map\(\) and Set\(\) \(and, of course, other collections or Enmaps\) cannot be inserted into the database.

Enmaps are a simple key/value pair storage, so there is no concept of a "column" here. Essentially, when inserting data, you're "setting" the value of a _key_ to a single _value_. Here is the simplest example possible:

```js
myTable.set("foo", "bar");
```

However, you can of course have something approximating rows by inserting either an array, or an object. For instance, here's how I do my per-server configuration for a bot:

```js
client.settings = new Enmap({name: 'settings', persistent: true});

const defaultSettings = {
  prefix: "!",
  modLogChannel: "mod-log",
  modRole: "Moderator",
  adminRole: "Administrator",
  welcomeMessage: "Say hello to {{user}}, everyone! We all need a warm welcome sometimes :D"
}

client.on("guildCreate", guild => {
  client.settings.set(guild.id, defaultSettings);
});
```

## Getting Data

Grabbing data from the PersistentCollection is just as simple as writing to it. You only need to know the key!

```js
myTable.get("foo"); // outputs "bar"
```

If your value is more complex, such as with guildSettings above, properties of the values are accessible as they would be normally. For instance:

```js
const thisConf = client.settings.get(message.guild.id);

if(!message.content.startsWith(thisConf.prefix)) return;
```

You could also access arrays in the same way. Here's an example, which also shows that this whole thing is syncronous:

```js
myTable.set("myArray", ["blah", "foo", "thingamajig", "goobbledigook"]);
myTable.get("myArray")[2]; // outputs "thingamajig"
```

## Editing Data

Enmap doesn't really have an "Edit" feature _per se_. You have two options when it comes to modifying existing data stored in Enmap.

First, you can simply set a new value, for simple keys:

```js
myTable.set("foo", "This is a new value!");
```

If the value stored in Enmap is either an Object or Array, you can use the "Prop" methods described below.

## Working with Properties

Starting with Enmap 2.0, new methods were introduced to work directly with properties. These methods only work with Object and Array values, which is what Enmap is very commonly used for. The following example describes how these properties are used:

```js
myTable.set("someObject", {
  firstprop: "myprop",
  currentPoints: 432,
  isActive: false,
  user: "139412744439988224"
});

// Get current points:
const points = myTable.getProp("someObject", "currentPoints"); // 432

// Modify the firstprop value
myTable.setProp("someObject", "firstprop", "New Value");

// Delete a property
myTable.deleteProp("someObject", "user");

// Check if a property exists
myTable.hasProp("someObject", "blah"); // false , since it doesn't have it.
```

## Some Use Cases

So, want to know what you can do with Enmap? Here's a couple of ideas for ya!

* **Per-Server Configuration**: As shown higher up. [A full example is available as a gist](https://gist.github.com/eslachance/5c539ccebde9fa76340fb5d54889aa22)
* **Tags**: Either per-server or global, they're simple strings so why not? I use them in my selfbot, [initializing in app.js](https://github.com/eslachance/evie.selfbot/blob/master/app.js#L14) and [controlling in a tags command](https://github.com/eslachance/evie.selfbot/blob/master/commands/tag.js).
* **Points system**: Adapted from the existing **JSON** and **SQLite** points system, [the Enmap version](/coding-guides/storing-data-in-a-persistent-collection.md) makes it even easier and cleaner than both of them!

## Multiple Persistent Enmaps

Using Enmap extensively for multiple things in your project? If you're using more than one, there's a method you can use that can initalize multiple Enmaps at once. This is a little advanced as far as code goes, but it does work fine:

```js
// As simple variables:
const Enmap = require('enmap');
const Provider = require('enmap-mongo');
const { settings, tags, blacklist } = Enmap.multi(['settings', 'tags', 'blacklist'], Provider, { url: config.mongodb.url });

// Attached to a Client object:
const Enmap = require("enmap");
const Provider = require("enmap-mongo");
Object.assign(client, Enmap.multi(["settings", "tags", "blacklist"], Provider, { url: client.config.mongo }));
```

> The 3rd argument for `multi()` is the "options", which are specific to each provider. In this case I'm giving a URL for mongodb to connect. Make sure to check the documentation for your provider to learn which options to use!

## Based on Collections

So, since Enmaps are based on [Discord.js Collections](/information/understanding-collections.md), it means they have almost all the Collection features you know and love. Want to grab all the guildSettings that have the default prefix? `client.settings.filter(c=>c.prefix === "!")`. Want to get all the tag names from a `tag` collection? `client.tags.map(t=>t.name).join(", ")`!