# Installing and Using Atom

Let's take a moment to appreciate the fact that the best code, is not just code that _works_ but also code that is _readable_. And _readable_ code is already easier to troubleshoot. To simplify the life of any code, a good editor is key. A good editor will tell you where your mistakes are, validate your code, give you best practices, and some will even run your code for you.

In this tutorial we'll be looking at one such editor: Atom. Now, I'm not personally biased towards or away from Atom, but the editor, and some help from people who use it, were readily available, so this is the one I'm showing you.

Other alternatives would be VS Code and Sublime Text 3. You'll have to look up specific instructions for those editors on your own.

## Getting Atom Installed

Far be it from me to actually show you how to install stuff on your computer - I will pretend for a moment you know what you're doing, and give you the outline:

1. Go to the [Atom.io website](https://atom.io/).
2. Click the big red **Download Windows Installer** button \(statistically, you're probably on Windows\).
3. Once the .exe is downloaded, run it, and install it.

Atom starts off with a simple interface with a nice welcome screen and a guide, so if you feel like it, go ahead and read up a bit.

## Opening your project

To the contrary of some more basic editors, Atom has the ability to open a project _folder_ so you don't have to keep opening files individually.

* Go to **File**, **Open Folder** \(CTRL+SHIFT+O\)
* Browse to the location of your main bot file \(mybot.js, app.js, or whatever\)
* Click on **Select Folder** \(_Note: your files will not appear here, only the folder structure. Don't worry they haven't gone anywhere_\)

On the left of the editor you will have all the files. Just double-click any of them to open it. To clean things up, you can close the Welcome Screen and such.

{% hint style="info" %}
Already, you can see that this code looks super clean, and colorful. Also the dark theme doesn't hurt the eyes so that's a plus.
{% endhint %}

## Getting ESLint installed

A 'Linter' is a plugin or app that verifies your code to tell you where the errors are, and also helps in formatting the code with proper indentation and styles.

{% hint style="info" %}
There's obviously a debate with what linter you should use. ESLint? JSHint? Some other obscure library? I'll use ESLint because... I know how to use it. Not endorsing it in any way!
{% endhint %}

ESLint works in 2 parts. The first is the plugin for Atom, which is installed through the command line:

* Hit Windows+R on your keyboard
* Type in `cmd` then press Enter
* Enter the command `apm install linter-eslint` then press Enter
* This may take a few seconds to a minute to complete, be patient.
* Keep the Command Prompt open.

Next, we need to install the eslint npm module, which is what actually verifies your code. `linter-eslint` just integrates those results in Atom.

In the command line, type `npm i -g eslint` then press Enter. Once it completes \(it will show you a list of eslint and installed dependencies\).

You'll need to restart Atom to take the changes into effect.

## Setting up ESLint options

ESLint, by default, will expect _very_ strict code from you, and will complain for a number of things. If you use the wrong type of quotes \(single vs double\), or if you use 4 instead of 2 spaces, alarms will go blaring.

ESLint options can be global or local to your project. Let's do a local one as an example.

* Create a new file in Atom
* Save it as `.eslintrc.json` in your project folder \(where your app.js/mybot.js is\)
* Copy the code below inside the file and save it again:

```javascript
{
    "env": {
        "es6": true,
        "node": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "sourceType": "module"
    },
    "rules": {
        "no-console": "off",
        "indent": [
            "error",
            2
        ],
        "linebreak-style": [
            "error",
            "windows"
        ],
        "quotes": [
            "warn",
            "double"
        ],
        "semi": [
            "warn",
            "always"
        ]
    }
}
```

Don't be daunted by this: it's just a little config in JSON format. I won't go into details of what each option does, they are all explained [in the docs](http://eslint.org/docs/rules/) but this is the rules I'm using. Basically:

* Force linter to support node
* Force linter to support ES6 code `<3`
* Indent to 2 spaces \(and not a 4-space-sized tab, ugh\)
* Make linebreaks Windows styles
* Warn on single quotes, prefer double quotes
* Warn when the semicolon is absent

