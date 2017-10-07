# Introducing Enhanced Maps

Enmap is a data structure that can be used to store data in memory that is also optionally saved in a database behind the scenes. When persistence is enabled, the data is synchronized to the database automatically, seamlessly, and asynchronously.

If persistence is turned off, Enmap acts as a regular Discord.js Collection object, with all the awesome features of Array Methods wrapped into Javascript's native Map() object.

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

## Caveat: Multithreading isn't available on Level

It is currently not possible to load a persistent LevelDB Enmap from multiple files - it *must* be loaded from a single location. For simple bots, you can easily just attach it to the client itself: 

```js
client.myTable = new Enmap({name: "myTable"});
```

This means wherever your `client` variable is available, so is the data in the Enmap!

Alternatively, the `enmap-rethink` provider works great in multi-process, or sharded, environments! 

```js
const EnmapRethink = require('enmap-rethink');
const provider = new EnmapRethink({name: "myTable"});
client.myTable = new Enmap({provider: provider});
```


## Inserting Data

Inserting data is super simple once you've initialized the table. Please note however that inserting data has its limits: 

* **Keys** must be either a string or a number. Any other value is rejected.
* **Values** must be simple data types on which JSON.stringify\(\) can be applied. Arrays, Objects, Strings, Numbers, those are fine. However, complex types like Map\(\) and Set\(\) \(and, of course, other collections or Enmaps\) cannot be inserted into the database. 

Enmaps are a simple key/value pair storage, so there is no concept of a "column" here. Essentially, when inserting data, you're "setting" the value of a _key_ to a single _value_. Here is the simplest example possible: 

```js
client.myTable.set("foo", "bar");
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
client.myTable.get("foo"); // outputs "bar"
```

If your value is more complex, such as with guildSettings above, properties of the values are accessible as they would be normally. For instance: 

```js
const thisConf = client.settings.get(message.guild.id);

if(!message.content.startsWith(thisConf.prefix)) return;
```

You could also access arrays in the same way. Here's an example, which also shows that this whole thing is syncronous: 

```js
client.myTable.set("myArray", ["blah", "foo", "thingamajig", "goobbledigook"]);
client.myTable.get("myArray")[2]; // outputs "thingamajig"
```

## Editing Data

There's no "edit" feature in Maps. Really what you need to do is to load the data, change it in memory, and re-write it. So, here's an example of loading a guild's settings, changing a value, and saving that change: 

```js
const thisConf = client.settings.get(message.guild.id);

thisConf.prefix = "+";

client.settings.set(message.guild.id, thisConf);
```



## Some Use Cases

So, want to know what you can do with PersistentCollection? Here's a couple of ideas for ya!

* **Per-Server Configuration**: As shown higher up. [A full example is available as a gist](https://gist.github.com/eslachance/5c539ccebde9fa76340fb5d54889aa22)
* **Tags**: Either per-server or global, they're simple strings so why not? I use them in my selfbot, [initializing in app.js](https://github.com/eslachance/evie.selfbot/blob/master/app.js#L14) and [controlling in a tags command](https://github.com/eslachance/evie.selfbot/blob/master/commands/tag.js).
* **Points system**: Adapted from the existing **JSON** and **SQLite** points system,[ the Enmap version](/coding-guides/storing-data-in-a-persistent-collection.md) makes it even easier and cleaner than both of them!

## Based on Collections

So, since Enmaps are based on Discord.js Collections, it means they have _all_ the Collection features you know and love. Want to grab all the guildSettings that have the default prefix? `client.settings.filter(c=>c.prefix === "!")`. Want to get all the tag names from a `tag` collection? `client.tags.map(t=>t.name).join(", ")` ! The possibilities are endless ^\_^

