# Using Switch Cases with Guidebot

This page is an explanation of how to use switch cases with a feature in Guidebot, `message.flags`.

## An Example

Switch cases are a nice feature that can be used without the `message.flags` feature, and can be used like this. 

```js
// This will look for the first argument. If your command was 'hi', and you did 'hi send', it would send 'Hi!' to the channel.
const trigger = args[0];

switch (trigger) {  
  case ('send'): {
    // This will send in the channel that the command was run in.
    message.channel.send('Hi!');
    break;
  }

  case ('dm'): {
    // This will DM the message author..
    message.author.send('Hi!');
  }
}
```

However, you can use the switch case with the neat little `message.flags` concept.

```js
  // This will look for the first argument beginning with -, the flag. If your command was 'hi', and you did 'hi -send', it would send 'Hi!' to the channel.
  switch (message.flags[0]) {
    case ('send'): {
    // This will send in the channel that the command was run in.
    message.channel.send('Hi!');
    break;
  }

  case ('dm'): {
    // This will DM the message author.
    message.author.send('Hi!');
  }
}
```