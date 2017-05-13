#Understanding Collections

In this page we will explore Collections, and how to use them to grab data from various part of the API.

A **Collection** is a *utility class* that stores data. In *Discord.js 9.x*, they extend the *Map* data type with additional helpers to assist in retrieving information from them.

Examples of Collections include:

- `client.users`, `client.guilds`, `client.channels`
- `guild.channels`, `guild.members`
- message logs (in the callback of `fetchMessages`)
- `client.emojis`

> In discord.js 8 and older, the equivalent was *Caches* which extended *Array*. Since *Collections* extend *Map* instead, **the methods have changed**.

## Getting by ID

Very simply, to get anything by ID you can use `Collection.get(id)`. For instance, getting a channel can be `client.channels.get('81385020756865024')`. Getting a user is also trivial: `client.users.get('139412744439988224')`

## Finding by key

If you don't have the ID but only some other property, you may use `find()` to search by property:

`let guild = client.guilds.find('name', 'Discord.js Official');`

## Custom filtering

*Collections* also have a custom way to filter their content with an anonymous function:

`let large_guilds = client.builds.filter(g => g.members.size > 100);`

## Other helpers

Some other helpers that are available in *Collections*:

- .get(id)
- .map(function)
- .find(key, value)
