---
layout: page
title: git
---

Setup
-----

    sudo apt-get install git bash-completion

Config
------

    git config --global user.name "Marchenko Alexandr"
    git config --global user.email "marchenko.alexandr@gmail.com"
    git config --global core.safecrlf false
    git config --global core.autocrlf true


Stage deleted files
-------------------

    git add -u

Remove submodule
----------------

Delete submodule from `.gitmodules`, `.git/config`, `.git/modules/[path to module]`

Logs
----

    git log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short

Git aliases
-----------

In `~/.gitconfig`

    [alias]
        co = checkout
        ci = commit
        st = status
        br = branch
        hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short

In `~/.bash_aliases`

    alias gs='git status '
    alias ga='git add '
    alias gb='git branch '
    alias gc='git commit'
    alias gd='git diff'
    alias go='git checkout '
    alias gk='gitk --all&'
    alias gx='gitx --all'

    alias got='git '
    alias get='git '

Revert local unstaged changes
-----------------------------

    git checkout hello.html

Unstage staged changes
----------------------

    git reset HEAD hello.html

Changes will be still here, to revert them use

    git checkout hello.html

Revert last commit
------------------

    git revert HEAD --no-edit

This will revert last commit, but commit will still be in history

Revert and remove commits
-------------------------

    git reset --hard v1 #or hash

Change previous commit
----------------------

    git add hello.html
    git commit --amend -m 'msg'

Create branch and switch to it
------------------------------

    git checkout -b style

Merge brachens
--------------

    git checkout [branch to update]
    git merge [branch to update from]

If there is conflicts while mergin - just fix and commit them

To list available local branches use:

    git branch

And to list available remote branches use:

    git branch -a

Add remote branch
-----------------

    git branch --track [local branch name] origin/[remote branch name]

Delete branch
-------------

    git branch -D gh-pages # deletes local branch
    git push origin --delete gh-pages # deletes remote branch

Remotes
-------

    git remote
    git remote show origin

Submodules
----------

To ignore `dirty` changes add `ignore = dirty` to problematic submodule in youd `.gitmodules` file

Update forkede repo
-------------------

    git remote add upstream https://github.com/acme/project.git
    git fetch upstream
    git checkout master
    git rebase upstream/master
    # - or -
    git merge upstream/master


Links
=====

http://githowto.com/ru/
