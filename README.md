# stash - A simple wrapper around stow and git for managing dotfiles

**Consider this a work in progress!**

Stash is a wrapper around stow and git to help manage dotfiles across multiple
machines.

The basic idea with stash is that you create a git repository with all the
dotfiles and config files that you want to have stow manage for your user
across multiple machines.

The "master" branch should be for configuration files that are common - files
that are common to all your systems (even if they will be customized per
system). Consider "master" as a starting place for customized configurations,
as well as configurations that don't vary by machine.

Stash will create a separate branch for every system it's used on. This branch
will be named with the system's hostname (specifically, "hostname -s"), and the
idea is that files in this branch are customized from the common defaults, and
can contain any addition files as well that are specific to the local host.

Stash will rebase the local hostname branch on "master", to integrate changes
in the upstream common version. Obviously, this can result in conflicts that
might have to be resolved with git mergetool.

Before using stash, you'll need to initialize a git repository (in
$STASH/$PACKAGE - see the stash script), add an initial set of common files,
and then commit to the "master" branch. All files in this repository will be
linked (by stow) relative to $TARGET (~ by default).

Stash only uses a single stow package for all files at this point.

## Configuration

Currently, stash configuration is done as variables in the stash script.
* STASH=~/stash - This is the directory that will be the top level stow
directory.
* PACKAGE=stow - This is the name of the stow package, and the directory that
contains your git repository of configuration files.
* TARGET=~ - This is the target directory for stow.
* HOST=$(hostname -s) - The name of the branch for the local host.

## Running stash

When run, stash will:

* Create a branch in $STASH/$PACKAGE for the local host, if necessary.
* Commit local updates to configuration files if there are local changes.
* Rebase the local branch onto any changes in "master".
* Run stow set up symbolic links.

## Caveats

The stow directory is linked by your dotfiles. Be careful if you switch
branches within the repository. Stash will attempt to checkout the local
host's branch when run, but if this might fail. Also, be aware that the
repository files are symlinked from your real dotfiles - this could result
in a broken or inconsistent state if you forget to check out the local branch
before logging out.

## Example

This is a rough outline of getting started with stash:

```
# First, lets get stash
$ cd ~
$ git clone https://github.com/scotte/stash.git
$ cd stash

# Next, lets create our git repository for stow
$ mkdir stow
$ cd stow
$ git init .

# Now, stage your initial set of configuration files
# mv files into ~/stash/stow...
$ git add ...
$ git commit -m'Initial commit'

# Note: you must remove all local files that will now be linked by stow.

# Finally, lets run stash for the local system
$ ~/stash/stash
Switched to a new branch 'my-desktop'
LINK: .xsession => stash/stow/.xsession
LINK: .Xdefaults => stash/stow/.Xdefaults
LINK: .zshrc => stash/stow/.zshrc
LINK: .bashrc => stash/stow/.bashrc
LINK: .vimrc => stash/stow/.vimrc
```

Now, I can update my local configuration files, customizing them for the local
system, then when ready to commit changes (and rebase on "master"), I'll run
stash again:

```
$ ~/stash/stash

# If there are local changes, you'll be tossed into $EDITOR for "git commit".
Current branch my-desktop is up to date.
```

That's it! If there were changes in "master", the local "my-desktop" branch
would have been rebased onto it. stow is also run to freshen links as required.
In this example, there were no changes to rebase from "master" and no updated
links.
