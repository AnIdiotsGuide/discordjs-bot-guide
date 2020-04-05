# Collections

In this page we will explore Collections, and how to use them to grab data from various part of the API.

A **Collection** is a _utility class_ that stores data. Collections are the Javascript Map\(\) data structure with additional utility methods. This is used throughout discord.js rather than Arrays for anything that has an ID, for significantly improved performance and ease-of-use.

Examples of Collections include:

* `client.users`, `client.guilds`, `client.channels`
* `guild.channels`, `guild.members`
* message logs \(in the callback of `fetchMessages`\)
* `client.emojis`

## Getting by ID

Very simply, to get anything by ID you can use `Collection.get(id)`. For instance, getting a channel can be `client.channels.get("81385020756865024")`. Getting a user is also trivial: `client.users.get("139412744439988224")`

## Finding by key

If you don't have the ID but only some other property, you may use `find()` to search by property:

`let guild = client.guilds.find(guild => guild.name === "Discord.js Official");`

The _first_ result that returns `true` within the function, will be returned. The generic idea of this is:

`let result = <Collection>.find(item => item.property === "a value")`

You can also be looking at other data, properties not a the top level, etc. Your imagination is the limit.

Want a great example? Here's getting the first role that matches one of 4 role names:

```javascript
const acceptedRoles = ["Mod", "Moderator", "Staff", "Mod Staff"];
const modRole = member.roles.find(role => acceptedRoles.includes(role.name));
if(!modRole) return "No role found";
```

{% hint style="info" %}
Don't need to return the actual role? `.some()` might be what you need. It's faster than find, but will only return a boolean true/false if it finds something:
{% endhint %}

```javascript
const hasModRole = member.roles.some(role => acceptedRoles.includes(role.name));
// hasModRole is boolean.
```

## Custom filtering

_Collections_ also have a custom way to filter their content with an anonymous function:

`let large_guilds = client.guilds.filter(g => g.memberCount > 100);`

`filter()` returns a new collection containing only items where the filter returned `true`, in this case guilds with more than 100 members.

## Mapping Fields

One great thing you can do with a collection is to grab specific data from it with `map()`, which is useful when listing stuff. `<Collection>.map()` takes a function which returns a string. Its result is an array of all the strings returned by each item. Here's an example: let's get a complete list of all the guilds a bot is in, by name:

```javascript
const guildNames = client.guilds.map(g => g.name).join("\n")
```

Since `.join()` is an array method, which links all entries together, we get a nice list of all guilds, with a line return between each. Neat!

We can also get a most custom string. Let's pretend the `user.tag` property doesn't exist, and we wanted to get all the user\#discrim in our bot. Here's how we'd do it \(using awesome template literals\):

```javascript
const tags = client.users.map(u=> `${u.username}#${u.discriminator}`).join(", ");
```

## Combining and Chaining

In a lot of cases you can definitely chain methods together for really clean code. For instance, this is a comma-delimited list of all the small guilds in a bot:

```javascript
const smallGuilds = client.guilds.filter(g => g.memberCount < 10).map(g => g.name).join("\n");
```

## More Data!

To see **all** of the Discord.js Collection Methods, please [refer to the docs](https://discord.js.org/#/docs/main/stable/class/Collection). Since Collection extends Map\(\), you will also need to refer to [this awesome mdn page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Map) which describe the native methods - most notably `.forEach()`, `.has()`, etc.

