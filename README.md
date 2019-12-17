# Git Process

The goal is to teach you how to arrive at a linear git history, which looks like this:

![linear git history](https://i.imgur.com/1W6bwyt.png)

Rather than something which looks like this:

![confusing git history](https://i.imgur.com/ye6ijuJ.png)

This post doesn't explain why this is helpful. It only shows you how to acheive it.

[Why keep a linear git history?](https://www.bitsnbites.eu/a-tidy-linear-git-history/)

## TL;DR

> every merge commit is a potential source of a non-linear history

Avoid merge commits.

[There's a more in-depth explanation of each of the following steps below this section](#beginning-new-features).

If you're just interested in getting started, here's what you need:

```bash
cd <my_project_repo>                                # move into your project

git branch --set-upstream-to=origin/master master   # keep local master in sync with
                                                    # origin/master (Github's copy of master)
                                                    # (only do this once per respository)

git checkout master                                 # switch to the master branch

git pull origin master                              # update your local copy of master to be
                                                    # Github's copy of master

git checkout -b <feature-branch-name>               # create a new branch for your feature

# 1. make changes
# 2. commit changes to feature-branch-name
# 3. repeat until your feature is done

# Ready to open a Pull Request?

git checkout feature-branch-name                    # make sure you're on your feature branch

git rebase -i head~<number-of-commits-you-made>     # squash your commits into a single commit

# An interactive rebase prompt opens in a file:
#   1. Change the first "pick" to "r"
#   2. Change all subsequent "pick"s to "f"s
#   3. Save and close the file

# A commit message prompt opens in a file:
#   1. Write the commit message for your feature's single commit
#   2. Save and close the file

git checkout master                                 # switch back to master

git pull origin master                              # update local copy of master to be
                                                    # Github's copy of master

git checkout <feature-branch-name>                  # switch back to your feature branch

git rebase master                                   # rebase your changes

# you may be done now if there were no rebase conflicts. Otherwise:

git status                                          # take a look at the rebase conflicts

# use your editor to resolve the conflicts

git add <filename-with-resolved-rebase-conflicts>   # mark the conflicting files as
git add <filename-with-resolved-rebase-conflicts>   # resolved

git rebase --continue

git push origin <feature-branch-name>               # push your branch up to Github

# open a pull request on github

# Ready to merge?

git checkout master
git merge feature-branch-name
git push origin master

```

## GITX

The best way to learn how Git works is by seeing the commit graph.

[Gitx is what I use to see git's commit graph](https://rowanj.github.io/gitx/).

If you want to try it:

1. [Download the package](http://builds.phere.net/GitX/development/GitX-dev.dmg)
2. Move it into your `/Applications` directory
3. Run it
4. Enable terminal usage:

![enable gitx terminal usage](https://i.imgur.com/ILQeNvj.png)

Now you can type the following within any git directory to see the commit graph.

```bash
gitx
```

## Beginning New Features

Let's use an example.

There's a story in Jira, titled:

> Allow doctors to login

To begin working on this feature, let's take a look at the current state of our git history:

![initial git history](https://i.imgur.com/pKmeyZn.png)

The first thing you should do when beginning a new feature is make sure you have the most up-to-date copy of `master`.

To do this, first run:

```bash
git branch --set-upstream-to=origin/master master
```

This tells git to keep your local copy of `master` in sync with the updated copy of `master` which you will pull down from `origin` (Github). You only need to run this once per Github repository, not every time you develop a new feature.

Now let's pull down an updated copy of `master` from Github:

```bash
git checkout master
git pull origin master
```

In our scenario, two commits had been added to `master` by our teammates since we last ran `git pull origin master` (SHAs: `6cdf6dd` & `a137df2`) and now since we've ran `git pull origin master` again, our local copy is up to date with what is on Github.

![updated master history](https://i.imgur.com/kv7s6gX.png)

Had we not told git to keep our local copy of `master` in sync with the Github copy of `master` by running `git branch --set-upstream-to=origin/master master`, your history would now look like this:

![out of sync master](https://i.imgur.com/GvcykMp.png)

This will lead to problems, so be sure to run `git branch --set-upstream-to=origin/master master`. (Again, you only need to run this once for each Github repository; not every time you begin a new feature).

Now let's make our new feature branch:

```bash
git checkout master             # point git's head at master
git checkout -b doctor-login    # create our feature branch
```

Now you can work on your feature, wip-committing as much as you'd like as you make progress.

## Finishing New Features

Let's say you've made 4 commits towards allowing doctors to login and you're ready to submit a pull request.

The first thing you'll want to do is _squash_ your work-in-progress commits down into a single commit.

### Squashing Commits

Each commit which ends up going into the `master` branch should contribute a single, atomic unit of business value and should typically correspond in a 1:1 way with a single story from Jira.

![feature complete](https://i.imgur.com/qxcF8a1.png)

To squash your 4 commits into a single commit, run:

```bash
git rebase -i head~4
```

Replacing `4` with however many commits you need to squash.

This will bring up an _interactive rebase prompt_ (used to squash commits), which looks like this:

![interactive rebase prompt](https://i.imgur.com/Z3gEiWN.png)

Change the first `pick` to an `r` and the remaining `pick`s to `f`s, like so:

![interactive rebase prompt complete](https://i.imgur.com/0HpHRqp.png)

Save and close the file.

This will automatically open a new file, where you're given the opportunity to write the commit message of the single, squashed commit which will go into the `master` branch when the pull request is merged:

![reword commit message prompt](https://i.imgur.com/kn3QRK2.png)

See [how to write a git commit messages](https://chris.beams.io/posts/git-commit/) for how to best write these.

We'll settle on this:

![reword commit message complete](https://i.imgur.com/mlJCHpb.png)

Save and close the file.

Your git history should look like this:

![squashed history](https://i.imgur.com/XnS1nlf.png)

We're almost done.

Before we open a pull request, let's make sure we're still working on top of the most up to date version of master:

```bash
git checkout master
git pull origin master
```

Now our history looks like this:

![unrebased history](https://i.imgur.com/uLRoEHG.png)

### Rebasing

It looks like our teammates have made several changes to master since we began working on our bigger doctor-login feature...

We want to change our commit's history to include all of master's history:

![unrebased history explanation](https://i.imgur.com/C3UIXOS.png)

This is when we _rebase_ our single commit onto `master`:

```bash
git checkout doctor-login
git rebase master
```

This is when you may encounter rebase conflicts if the team changed any of the same spots in any of the files which you made changes to in their recent commits in master:

![rebase conflict](https://i.imgur.com/wIGvdU3.png)

Let's open the conflicting file and resolve the conflictions:

![conflicting file content](https://i.imgur.com/MRgSRrM.png)

The most up-to-date version of `master` is showing the version as `1.16.11` and we want our change to bump the version as well, so let's go with this resolved content:

![conflicting file resolution](https://i.imgur.com/oilpxZB.png)

Save and close the file.

```bash
git add pro/__init__.py
git rebase --continue
```

You're all set!

![squashed and rebased history](https://i.imgur.com/1wxtcqU.png)

Push up your branch and open a pull request.

### Merging into master

```bash
git checkout master
git merge doctor-login
git push origin master
```

The pull request page will update to indicate that your branch was merged!

Here's your beautifully updated git history:

![completed history](https://i.imgur.com/CRJbKNT.png)
