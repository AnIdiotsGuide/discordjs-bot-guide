# PersistentCollection Guide

Persistent Collections are a data structure that can be used to store data in memory that is also saved in a database behind the scenes. The data is synchronized to the database automatically, seamlessly, and asynchronously so it should not adversely affect your performance compared to regular  Discord.js Collections

So why use this when you have JSON and SQLite? Because with a PersistentCollection you don't have to write any code to read or write from files, no SQL queries, no complications. You just insert data inside a Collection, and it's saved in the database. That's it!

## Installing

PersistentCollections can be installed from `npm`:

```js
npm install --save djs-collection-persistent
```

They can be imported into your bot code using the following:

```js
const PersistentCollection = require('djs-collection-persistent');
```



## Creating a "Table"

Most of us understand the concept of a "table", basically a structure that holds multiple rows and columns. This applies to concepts from Databases to Excel Spreadsheets so to simplify this how-to I'll go ahead and use those terms.

To create a new table, then, you need to "initialize" a new PersistentCollection:

```js
const myTable = new PersistentCollection({name: "myTable"});
```

When this code is run, one of 2 things can happen:

* **If there is no table of that name**, this table is created in the database, and it's empty.
* **If the table exists**, that table and _all its contents_ is loaded into the Collection itself. 

So what does this mean? It means that "initializing the database" and "loading all its data" is taken care of, for you. You don't need to do anything else for this to happen.

> While it's possible to put the initialization line in multiple files, it's recommended not to do so as each load duplicates the whole data set into memory. That is to say, the more files load a table, the more memory you're taking. To prevent that, simply attach the PersistentCollection to your client variable: `client.myTable = myTable;` for instance.



## Inserting Data

Inserting data is super simple once you've initialized the table. Please note however that inserting data has its limits: 

* **Keys** must be either a string or a number. Any other value is rejected.
* **Values** must be simple data types on which JSON.stringify\(\) can be applied. Arrays, Objects, Strings, Numbers, those are fine. However, complex types like Map\(\) and Set\(\) \(and, of course, other collections\) cannot be inserted into the database. 

Collections are a simple key/value pair storage, so there is no concept of a "column" here. Essentially, when inserting data, you're "setting" the value of a _key_ to a single _value_. Here is the simplest example possible: 

```js
myTable.set("foo", "bar");
```

However, you can of course have something approximating rows by inserting either an array, or an object. For instance, here's how I do my per-server configuration for a bot: 

```js
const guildSettings = new PersistentCollection({name: 'guildSettings'});

const defaultSettings = {
  prefix: "!",
  modLogChannel: "mod-log",
  modRole: "Moderator",
  adminRole: "Administrator",
  welcomeMessage: "Say hello to {{user}}, everyone! We all need a warm welcome sometimes :D"
}

client.on("guildCreate", guild => {
  guildSettings.set(guild.id, defaultSettings);
});
```



## Getting Data

Grabbing data from the PersistentCollection is just as simple as writing to it. You only need to know the key!

```js
myTable.get("foo"); // outputs "bar"
```

If your value is more complex, such as with guildSettings above, properties of the values are accessible as they would be normally. For instance: 

```js
const thisConf = guildSettings.get(message.guild.id);

const prefix = thisConf.prefix;

if(!message.content.startsWith(prefix)) return;
```

You could also access arrays in the same way. Here's an example, which also shows that this whole thing is syncronous: 

```js
myTable.set("myArray", ["blah", "foo", "thingamajig", "goobbledigook"]);

myTable.get("myArray")[2]; // outputs "thingamajig"
```



## Editing Data

There's no "edit" feature in Maps. Really what you need to do is to load the data, change it in memory, and re-write it. So, here's an example of loading a guild's settings, changing a value, and saving that change: 

```js
const thisConf = guildSettings.get(message.guild.id);

thisConf.prefix = "+";

guildSettings.set(message.guild.id, thisConf);
```



## Some Use Cases

So, want to know what you can do with PersistentCollection? Here's a couple of ideas for ya!

* **Per-Server Configuration**: As shown higher up. [A full example is available as a gist](https://gist.github.com/eslachance/5c539ccebde9fa76340fb5d54889aa22)
* **Tags**: Either per-server or global, they're simple strings so why not? I use them in my selfbot, [initializing in app.js](https://github.com/eslachance/evie.selfbot/blob/master/app.js#L14) and [controlling in a tags command](https://github.com/eslachance/evie.selfbot/blob/master/commands/tag.js).
* **Points system**: Both "JSON" and "SQLite" tutorials use this as an example, it could easily be adapted with PersistentCollection. You could load points with `points.get(message.author)` for example. 

## These are Collections after all

So, since PersistentCollection extends Collection, it means that _all_ the Collection features you know and love. Want to grab all the guildSettings that have the default prefix? `guildSettings.filter(c=>c.prefix === "!")`. Want to get all the tag names from a `tag` collection? `myTags.map(t=>t.name).join(", ")` ! The possibilities are endless ^\_^

