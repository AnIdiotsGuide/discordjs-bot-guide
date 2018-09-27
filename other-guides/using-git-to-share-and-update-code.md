# Using Git to share and update code

Have you ever come to a point where you're editing code, removing and adding and changing stuff and all of a sudden you realize, _Shit, I deleted this piece and I need to rewrite or re-use it now. Damn!_

Have you ever wished there was a simpler way to transfer your code to your hosting, rather than having to connect to an FTP, or zip your file, upload to your host and unzip... you know the drill, right? Ugh!

Have you ever wished you could easily share your code and have other people help you out in some bits, making your bot better?

If you answered **Yes** to any of these questions, boy do I have a product for _you_! It's called `git` and it's a pretty magical tool.

## Pre-requisites and Software

To take full advantage of `git` you need to first have _at least_ completed the [Adding a config.json file](../first-bot/your-first-bot.md#adding-a-configjson-file-to-your-bot) walkthrough. This means that at the very least, your token is located in a separate file, and not within your javascript files. Having the prefix and the owner ID in that separate configuration file means that you get the advantage of easy testing: Change the prefix and token, and you have a secondary bot you can test your code with, wooh!

What else do you need? `git` itself, of course! For Windows get [Git SCM](https://git-scm.com/download/win) , on Linux run `sudo apt-get install git-all` or `sudo yum install git-all` depending on your distro's install method. Mac users also have a [Git SCM](http://git-scm.com/download/mac) installer.

Anything else? Eeeeeh, nope! That's all, really. Let's get on to usage!

## Initialization and `.gitignore`

So the first baby step into the world of git, is to initialize your project folder as a git repository. To do that, you need to navigate to that folder in your command line. Or use an OS shortcut:

* Mac users can use [this trick from lifehacker.com](http://lifehacker.com/launch-an-os-x-terminal-window-from-a-specific-folder-1466745514).
* Windows users, remember the magic trick: SHIFT+Right-Click in your project folder, then choose **Open command window here**.
* Most Linux distros have an **Open in Terminal** option. But you use Linux, you can figure it out, right?

Once you have your prompt open in that folder, go ahead and run `git init`. It doesn't have any options or questions - it just inits the folder.

### Ignoring Files

In git, one of the most important files is `.gitignore` which, as the name would imply, ignores certain files. This is pretty critical with node.js apps, and our bot: you definitely want and need to ignore both the `node_modules` folder and the `config.json` file. The former because you don't want thousands of files being uploaded \(and they don't work on other systems anyway\), the latter because you don't want your token to end up on a public `git` repository!

There are pre-built `.gitignore` files you can grab off the internet, but for the purpose of this exercise you only need 2 lines in that file:

```text
node_modules
config.json
```

But how do you create it? Linux and Mac users, you can probably easily create the file directly. Windows users, you have to take a tiny detour - Windows doesn't like files with only an extension and no filename. Don't worry thought it's simple. Start by creating a new file called, for example, `gitignore.txt` and put in the 2 lines above. Then, in your console, run `rename gitignore.txt .gitignore` and this will rename it as expected.

## Your first `git` command

And now we're ready to start to `git gud` \(no no don't type that it's a meme!\)

The first command you can try is `git status`. This will show you a list of all your files and in all subfolders and indicate them as **untracked files**, meaning they're not being tracked \(saved\) by git.

Let's resolve that with our second command, which essentially tells git to track all these files and start doing its magic on them: `git add .` where `.` is just a shortcut to 'all the files including subfolders'.

And now it's time to _commit_ to the task. _Commiting_ basically means you're creating a snapshot of your project, that is forever saved in your history. A commit can be a small as a tiny bugfix of one line or rewriting a complete function or module. Committing is always going to be with a message that describes the change, so smaller commits can be easier to manage and track in the future. So here goes:

`git commit -m 'Initial Project Commit'`

This will output a new commit with some stats including the number of changed lines, and the list of changed files. Great! Now we've got a commit. Funny story: you can continue committing changes locally, and do all the great things `git` does without every pushing to a remote repository. But that's what we're here for, right? So let's move on!

## Creating a GitHub/GitLab/BitBucket account

With `git` you have a lot of choices when it comes to hosting your projects online. Pushing to an online repository gives you the safety in backups - your whole commit history is available, and nothing is lost whatever happens. Also gives you the ability to share your projects either publicly or privately with other people, and have them contribute to your project if you want them to.

The 3 main `git` hosting services that are free are GitHub, GitLab and BitBucket. While all three are free, GitHub is the only one that doesn't offer private repository - but it's also the most _stable_ and _open_. I'll be going with GitHub in this case, but most of the steps I'll take here are the same for all 3 services. You just need an account and you're good to go!

So let's go ahead and prepare GitHub. On [GitHub.com](https://github.com/) , create an account and then create a **New Repository**. Give it a name that identifies your project - it doesn't need to be identical to your folder name, and you can change it later if you want. Don't add a readme for now.

In the page that displays after creating the repository you actually get the instructions you need to do, under **push an existing repository from the command line**:

```text
git remote add origin git@github.com:your-username/your-repo-name.git
git push -u origin master
```

Obviously, change `your-username` and `your-repo-name` to the appropriate strings. The first line _links_ your local repository to the GitHub website. The second line actually takes all your local commit history, and it _pushes_ it directly to github. Note that this action will ask you to enter your github username and password.

'Wait, really? That's it?' you ask. Yep, your code is now on github. Refresh the github page and you'll see all your code there - without the `node_modules` and `config.json` if you've followed correctly!

## Pushing further updates

Linking your account doesn't save you from future commands. Actually, every time you make a change and want to push it to the repository you have to use that `push` command again. Note that you don't _need_ to push every commit individually - you can work for a day and do multiple commits, and push them all at once at the end of the day.

A small shortcut that you can use is to `add` and `commit` at the same time. This only works if you have changed files but not created new ones \(for that just do `git add .` separately\). So if you're done just small changes, you can use this:

```text
git commit -am 'Fixed permission issue in 8ball command'
```

And don't forget to `git push origin master` again!

## Getting your code to another machine

So, let's say you have a VPS hosting somewhere. Or a Raspberry Pi. Or whatever other machine. You want to get that code of yours onto it. You first need to follow the `install` steps above to get `git` installed on the machine, and then you need to _clone_ the repository from the git remote:

```text
git clone git@github.com:your-username/your-repo-name.git
```

That's all! Well, you need to enter your username and password again, but once that's done, that code is available!

Don't forget that any file that's been added to your `.gitignore` file isn't present here, so you'll need to do 2 things:

1. Create your `config.json` again, either with the same token and prefix, or different ones.
2. Install your node modules again. If you've properly run `npm init` and installed all your modules with the `-S` or `--save` argument, you should only need to do `npm install`. Otherwise, you need to install all of them again, for example using `npm install discord.js`

Now, remember that these files aren't synced so they won't be overwritten or modified whenever you push or pull to and from the repo.

Speaking of which, any time you have `push`ed to GitHub, you can get an updated copy of the code by simply running `git pull origin master`.

You'll have to reset your bot for the changes to take effect, of course.

## Some Last Tips

The process isn't one-way. If you modify files on _any_ computer you can `git push origin master` and `git pull origin master` on the other one. Be careful about editing on both locations though, as this may cause conflicts that are annoying to resolve \(and are beyond this guide's scope\)

IF you ever have 'temporary' changes and you want to overwrite them - like a quick path on your VPS that you've also fixed locally - you can avoid the whole 'conflict' process by _Resetting_ a repository to your last `pull`. Simply run `git reset --hard HEAD` to erase any local changes, then run `git pull origin master` to grab the latest changes.

You can connect to multiple remote repositories by running the `remote add` command above. You'll need a different name, for example instead of `origin` you can call a remote `gitlab` and then any command should reflect that, like `git pull gitlab master`.

There is a **lot** more to `git` than what was shown here \(and I'm aware even that's already a shitton of information\). You can create and merge `branches`, `revert` to previous commits, make and accept `PR`s... [There is a lot more](https://www.google.com/search?q=git+tutorials) that you can you can learn!

But for now... I think this massive wall of text is plenty.

