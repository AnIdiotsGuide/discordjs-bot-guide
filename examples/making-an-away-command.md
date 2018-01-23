# Making An Away Command
Sometimes, when a user goes AFK, they get pinged, but the person who pinged them never knows the reason why. Creating an away command can be used to notify the user the reason or if someone is away when they get pinged.
This is a greate and unique feature which can be implimented in a discord.js bot. If you follow this tutorial, you will successfully have an away command!

## Creating The Command
First of all, we will start by making a new Discord Collection
```js
const Discord = require('discord.js');
const away = new Discord.Collection();
```

Now lets make the away command.
```js

const prefix = '/';
if (message.content.startsWith(prefix + 'away')) {

}
```
This checks if the message starts with '/away' and if it does, it'll continue with the rest of the code.

```js
if (message.content.startsWith(prefix + 'away')) {
    const reason = args.join(' ');
    if (!away.has(message.author.id)) {
        away.set(message.author.id, reason);
    }
}
```
We then check whether the user's id is not in the collection, and if it is not then we will add the ID and the reason to the collection. To define reason it to the collection, take a look at [Command with arguments](./command-with-arguments.md).

```js
if(message.content.startsWith(prefix + 'away')) {
    const reason = args.join(' ');
    if (!away.has(message.author.id)) {
        away.set(message.author.id, reason);
    }
    
    if (away.has(message.author.id)) {
        away.delete(message.author.id);
    }
}
```

We'll now check if the user is already away, because if they are away, we want them to be able to remove their 'away' state. if the away collection has the user's ID, it'll remove it from the collection.

```js
if (message.content.startsWith(prefix + 'away')) {
    const reason = args.join(' ');
    if (!away.has(message.author.id)) {
        away.set(message.author.id, reason);
        return message.channel.send(`You have been set as away for the reason: ${reason}`);
    }
    
    if (away.has(message.author.id)) {
        away.delete(message.author.id);
        return message.channel.send('You are no longer away.');
    }
}
```

This is the end of our `away` command. Now we need to check whether the user is mentioned. We can do this by creating an array from the keys of the away collection. This should be at the top of our message event.

```js
client.on('message', message => {
    for (const user of away.keys()) {
    
    };
```

We then do a for loop in the new collection we've created. This allows us to check the ID's of each away person. 

```js
client.on('message', message => {
    for (const user of away.keys()); {
        if (message.mentions.has(user)) {
                return message.channel.send(`This user is currenty away for reason: ${away.get(user)}`);
        }
    };      
});
```

Now, we check if there is a message, and if the mention is the user who has set their status as 'away', then it'll send a message saying that they're away and the reason for it.


## Final code:

It's normal if your code doesn't look like this. Everyone makes mistakes. Hopefully this guide has helped you create a new command which helps makes your bot stand out.

```js
client.on('message', message => {
    let args = message.content.split(' '); //split the message per space
    args = args.splice(1) //remove the first word, i.e the command.
    for (const user of away.keys()) {
        if (message.mentions.has(user)) {
            return message.channel.send(`This user is currenty away for reason: ${away.get(user)}`);
        }
    };

    if (message.content.startsWith(prefix + 'away')) {
        const reason = args.join(' ');
        if (!away.has(message.author.id)) {
            away.set(message.author.id, reason);
            return message.channel.send(`You have been set as away for the reason: ${reason}`);
        }
            
        if (away.has(message.author.id)) {
            away.delete(message.author.id);
            return message.channel.send('You are no longer away.');
        }
    }
});
```
