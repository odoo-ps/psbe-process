## Introduction
Git is a Version Control System, it records changes made to the files in your project in 'commits',
snapshots of the state of the project.  
This allows you to keep a history of the development, to go back in time and check when this
bug/feature was introduced, to share your code and easily work with other contributors.

## Setup

##### Installing on Linux
If you're on Ubuntu, try `apt` :
    ```sh
    sudo apt install git
    ```
More details on other options can be found [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

##### Configuration

####### See [this](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) for Windows config

Now you'll want to set up git configuration. The global config file usually lives in `~/.gitconfig`
for your user. You can download this copy and adapt it for you. 
It also contains aliases and parameter we're going to go through later.

Alternatively you can run these to set your email and name.
Git is going to use them to define the author of the changes you introduce.
    ```sh
    git config --global user.name "John Doe"
    git config --global user.email johndoe@example.com
    ```

Git sometimes need text input and is going to open an editor for that.
If you want Git to use another editor than your system default one, vim for example, you can run :
    ```sh
    git config --global core.editor emacs
    ```

##### Github configuration
Odoo code lives in various repositories in on Github, you'll need to log in to pull and push.
If you want git to stop asking your password, you should [use ssh keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)  
Note that it's only working when using SSH. In short, when cloning using the SSH url `git://..` instead of the HTTPS one.

##### Extra tools
In the following, we're going to go over the usage of Git from the command line.
It's where you have the more options to work with and a good place to start learning git.
It's also probably not the fastest, most user-friendly. 
You might want to have a look at what's possible with you IDE or [git gui](https://git-scm.com/docs/git-gui/) [lazygit](https://github.com/jesseduffield/lazygit)

## Use
##### Start
To have git start tracking the files in a directory, you can use `git init`.
It will initialize a git repository in the current folder.
Otherwise, you can clone an existing repository with
// TODO update rep
```sh
git clone https://github.com/libgit2/libgit2
```

##### Committing
When in a git repository, files can be tracked or untracked. Git only bothers itself with tracked files.
Tracked files themselves can be in different states, unmodified, modified, staged.
In a usual wokflow, once we edited files, we stage those we want to commit with `git add path/to/file`.
Then running `git commit` makes a commit.
It open an editor for you to fill in the commit message
(see [guidelines](https://www.odoo.com/documentation/14.0/developer/misc/other/guidelines.html#git) for commit messages)
and when you close it, appends the new commit after the current one in the history tree.
All the staged files are now unmodified (since this last commit).

| ![Relation between files states](https://git-scm.com/book/en/v2/images/lifecycle.png) |
|:--:|
| From [git-scm](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository) |

##### Collaborating
After some development you end up with a chain of successives commits, unfortunately the other contributor too, on a
diverging branch, on their local repository
To figure out how to solve that, I'm inviting you to go through the *Introduction sequence* in **Main** and then the *Push & Pull* of
the **Remote** tab on (https://learngitbranching.js.org/)

#### Opening a PR
You've probably already read [Create your first PR]
(https://www.odoo.com/documentation/14.0/developer/howtos/rdtraining/16_guidelines_pr.html#create-your-first-pr)
from the training. 
There is a some differences in PS-Tech though.
 - the features branch PR usually target a `staging` instead of a `main` branch.
 - You shouldn't add a commit to the `staging` if there is already some commits to merge into the `main`
 - We're supposed to squash our development commits into one commit per feature.
It sometimes makes sense to keep several commits, if the development covers more than one module for example.

If your PR contains multiple commits, this can be relevant too to help with squashing
(https://github.com/odoo/enterprise/wiki/GIT-Cheatsheet#pull-request-flow).

After your PR has been reviewed, changes made and merged, it's a good idea to remove the now useless feature branch.
Github has a convenient button for it on the PR page. This command would also do it.
```sh
git push origin --delete 15.0-feature-tri
```

## Good practices

##### Rebasing
At Odoo, we choose a rebase workflow, it means you should restrain from using `merge` in favor of `rebase`.
This allows to have more linear, readable, history tree.


In order to follow this guideline might also want to change `git pull` default behavior,
forcing it to rebase instead of merging with :
```sh
git config --global pull.rebase true
```

##### Branches
When several developers work on a feature, they would usually have a public branch, say `15.0-feature`.
Then each contributor work on their private branch, `15.0-feature-tri`,
and rebase on the public branch to merge their work with the others.

##### Rewriting history
Rebasing comes with caveats though,
if your branch has been published on the remote and other contributors are based on it,
it's preferable not to rebase it. As you may have noticed in the tutorial, rebasing actually dupplicates commit.  
If a public branch is rebased the other contributors are still working on top of the old unrebased commits and it may
cause hard to solve merge conflict farther down the line.

This applies to amending, rewording commits as well, in short anything that rewrites history.
An indication that the history has been rewritten is that some commit now have a different hash.

In some cases rebasing doesn't actually rewrite history though,
if no commits are modified (not amended and no ancestor changes),
there is no need to. Git may simply fast-forward the branch pointer if needed.

If you rewrite history of a pushed branch, you'll probably have to force push or git will reject the push.
Now the issue again here is if someone else updated the same branch with their own commits,
then your force push is going remove them from the branch and you might not even notice it.
A solution is to use `git push --force-with-lease` instead. This will create a useful alias.
```sh
git config --global alias.fpush 'push --force-with-lease'
``` 

##### From staging to main
The staging branch being a public branch with other contributors based on it, it shouldn't be rebased.
Two options are :
 - if the staging contains has only one commit to merge to a `main` branch, which it probably should,
it's possible to open a PR and use the `Squash and merge` option from Github.
For some reason `Rebase and merge` won't fast-forward and will always rewrite history, so it shouldn't be used. 
 - Fast-forwarding `main` to `staging` in the CLI
 - ^  TODO : clarify, multiple staging


## Odoo notes
 - webhooks

## Guidelines ?
 - rebasing or merging fastforwarding
 - squashing, rewriting history
 - precommit

## Tips
reflog backup branch


## Going further
The rest of the exercises from https://learngitbranching.js.org/
Also see the game: https://ohmygit.org/
Git rebase in depth: https://git-rebase.io/
Man pages: https://git-scm.com/docs
And the book: https://git-scm.com/book/

Odoo specific scenarii
 - staging
 conflict fixing
 squashing
 splitting commit in two
 

