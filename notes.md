# Introduction
Git is a Version Control System, it records changes made to the files in your project in 'commits',
snapshots of the state of the project.  
This allows you to keep a history of the development, to go back in time and check when this
bug/feature was introduced, to share your code and easily work with other contributors.

# Setup

#### Installing on Linux
If you're on Ubuntu, try `apt`:

```sh
sudo apt install git
```

More details on other options can be found [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

#### Configuration

*See [this](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) for Windows config*

Now you'll want to set up git configuration. The global config file usually lives in `~/.gitconfig`
for your user. You can download [this copy](#gitconfig) and adapt it for you. 
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
git config --global core.editor vim
```

#### Github configuration
Odoo code lives in various repositories in on Github, you'll need to log in to pull and push.
If you want git to stop asking your password, you should [use ssh keys.](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)  
Note that it's only working when using SSH. In short, when cloning using the SSH url `git://..` instead of the HTTPS one.

#### Extra tools
In the following, we're going to go over the usage of Git from the command line.
It's where you have the more options to work with and a good place to start learning git.
It's also probably not the fastest, most user-friendly. 
You might want to have a look at what's possible with you IDE or
[git gui](https://git-scm.com/docs/git-gui/),
[lazygit](https://github.com/jesseduffield/lazygit).

# Use
#### Start
To have git start tracking the files in a directory, you can use `git init`.
It will initialize a git repository in the current folder.
Otherwise, you can clone an existing repository with
```sh
git clone https://github.com/libgit2/libgit2
```

#### Committing
When in a git repository, files can be tracked or untracked. Git only bothers itself with tracked files.
Tracked files themselves can be in different states, unmodified, modified, staged.
In a usual wokflow, once we edited files, we stage those we want to commit with `git add path/to/file`.

Then running `git commit` makes a commit.
It open an editor for you to fill in the commit message
(see [guidelines](https://www.odoo.com/documentation/14.0/developer/misc/other/guidelines.html#git) for commit messages)
and when you close it, appends the new commit after the current one in the history tree.  
All the staged files are now unmodified (since this last commit).

|![Relation between files states](https://git-scm.com/book/en/v2/images/lifecycle.png)|
| :----: |
| From [git-scm](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository) |

#### Branching
After some development you end up with a chain of successives commits.
The other contributor have a similar diverging branch, on their local repository.
To figure out how to solve that,
I'm inviting you to go through the *Introduction sequence* in **Main** and then the *Push & Pull* of
the **Remote** tab on <https://learngitbranching.js.org/>

# Odoo workflow

## Rebasing
At Odoo, we choose a rebase workflow, it means you should restrain from using `merge` in favor of `rebase`.
This allows to have more linear, readable, history tree.


In order to follow this guideline might also want to change `git pull` default behavior,
forcing it to rebase instead of merging with :
```sh
git config --global pull.rebase true
```

### Branches
In Odoo (and in many other places), some branches have special use and characteristics.  
You'll have to work with:

#### Personal feature branche
Branch containing your code. Usually named *odoo-version*\_*feature*\_*trigramme*.
That's where you'll commit most if not all of your code.
Once you have something that's worth sharing with the others, you can rebase it to the shared feature branch.

```sh
git switch 15.0-feature-tri
git rebase 15.0-feature-tri 15.0-feature
```

#### Shared feature branche
It's the branch containing all the code for a feature, yours and other contributors.
Usually named *odoo-version*\_*feature*.

Once the feature is ready it's should be reviewed before going on preproduction.
In order to do that, you should open a PR targeting the preproduction branch and add a reviewer.
You've probably already read
[Create your first PR](https://www.odoo.com/documentation/14.0/developer/howtos/rdtraining/16_guidelines_pr.html#create-your-first-pr)
from the training, if not you can refer to it.

Before **rebasing and merging** makes sure:
- The PR has been approved
- You squashed the development commits into one commit per feature, see (https://git-rebase.io/#squash),
except in some cases where it makes sense keep several commits.
- You check the other guidelines in the PR template


After your PR has been reviewed, corrected and merged, it's a good idea to remove the now useless feature branch.
Github has a convenient button for it on the PR page. This command would also do it.
```sh
git push origin --delete 15.0-feature
git push origin --delete 15.0-feature-tri
```

#### Production and preproduction branches
Usually `main` is the production branch, the code running on the client Odoo instance.
It may be behind `preproduction` by a few commits and **shouldn't diverge from it**,
that is to say `preproduction` is a direct descendant of `main`.
If it's not the case, it's a recipe for bad surprises when pushing code to production.


The extra commits in the preproduction branch contains the development being tested before being pushed in production.
These branches are sometime write protected and in any case you probably shouln't push unreviewed code on them.

Two options to push in production:
##### Github PR
If you have only one commit to merge is to open a PR and use the `Squash and merge` option from Github.
##### Fast-forwarding it.
Again, for this to work `preproduction` and `main` shouldn't be diverging.
If you're unsure test it locally without pushing, Git should tell you if it uses fast-forward. 
```sh
git rebase preproduction main
```
 
**What not to do :**
For some reason `Rebase and merge` won't fast-forward and will always rewrite history, so it shouldn't be used.

### Branches on Odoo.sh
If you're working on a project on Odoo.sh, you'll notice it lists the git branches under different categories.

##### Production
Again, it's the code in production.

##### Staging
These are branches that contains production data and some config, intended for testing.
Usually the preproduction branch will also be a staged branch.
Some feature branch might be staged during the development as well to get feedback from BAs or the client.

Each time commits are pushed to a staging, Odoo.sh will update the staging thanks to a github Webhook

##### Development
The rest of the branches. Odoo.sh will run tests on them for each push

# Warnings

#### Reformatting

Reformatting whole files or code you're not writing will wreck the history and is a sure way to trigger merge conflicts
with other contributors 

#### Rebasing and rewriting history
Rebasing comes with caveats,
if your branch has been published on the remote and other contributors are based on it,
it's preferable not to rebase it. As you may have noticed in the branching tutorial, rebasing actually dupplicates commits.  
If a public branch is rebased the other contributors are still working on top of the old unrebased commits and it may
cause hard to solve merge conflict farther down the line.

This applies to amending and rewording commits as well, in short anything that rewrites history.
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


If you follow the workflow as described above, it shouldn't be an issue:
- Rebasing is fair game on your private feature branch, no one should be developing on it
- Rebasing a feature branch at the end of development to preproduction is okay too,
as the feature is finished, no one should still be working on it.
- Rebasing the preproduction branch is okay as it's actually fast-forwarding.

#### Making a mess
It's hard to make something totally irrecuperable with git.
Even if you're unsure feel free to experiment as long as you don't push on the remote.

It is easier to go back to a previous state if you
[create a backup branch](https://docs.gitlab.com/ee/topics/git/git_rebase.html#before-rebasing)
before attempting a merge or a rebase.
That being said, even if you forget to do it,
and things go wrong you can probably retrieve the commits you're looking for with `git reflog`.

# Further reading
- The rest of the exercises from <https://learngitbranching.js.org/>
- Also see the game: <https://ohmygit.org/>
- Git rebase in depth: <https://git-rebase.io/>
- Man pages: <https://git-scm.com/docs/>
- And the book: <https://git-scm.com/book/>

## gitconfig
```
[user]
    email = tri@odoo.com
    name = Surname Name (tri)
[alias]
    g = grep --break --heading --line-number
    lg = log --graph --pretty=tformat:'%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%an %ar)%Creset'
    hist = log --graph --date=short --pretty=tformat:'%Cred%h%Creset %Cgreen%ad%Creset | %C(auto)%d%Creset %s %Cgreen(%an)%Creset' --all
    fpush = push --force-with-lease
[pull]
    rebase = true
```
 

