---
title: git like it's svn
layout: post
excerpt: Very svn-like git workflow
---

# git like it's svn

Here we are going for a work model that is as close to svn as
possible. This means that we never keep changes stages (we need to stage
for additions and removals), so we either only have unstaged changes,
or none at all. Also, we don't keep commit locally. Neither would even
be possible in svn.

## `svn checkout https://server/path/repo`

    git clone https://server/path/repo

By this you check out the default branch of the server repo, in contrast
to svn where the branch you get is determined by the exact path you use.

## `svn update`

    git stash
    git pull [--rebase]
    git stash pop

Stashing and unstashing are only needed when you have unstaged
changes. Whether you rebase here doesn't matter as you never have
unpushed commits.

This is also the only place where you can encounter conflicts. `svn
update` will just merge incoming changes into you locally modified files,
without giving you a way to undo this.

With git we turn this around, and first save the local changes, then
bring in the incoming changes. This can't cause conflicts as we don't
have local changes or local commits at this point. Conflicts are only
possible in the `git stash pop` part.

For all conflicted file (`git status` shows those), first resolve the
conflict, then do a `git add file`. When no more conflicts remain, do
a `git reset` (this step is necessary to get the changes out of the
staging area, so they are unstaged changes again, as needed for the
working model of this section).

What kind of conflicts you get here is obviously only dependend on
your and you collaborator's work, not on the version control system you
are using.

## `svn commit -m 'msg' files`

    git commit -m 'msg' files
    git push

This version works as long as no files are new or deleted, then we
can skip the staging area completely. Also, `svn commit` ensures that
the commit is either happening or not (in case a `svn update` must be
performed first). In git, we can't do the two commands atomically, so
in case the push fails we need to first do a `git reset HEAD^` to undo
the commit. Then the `svn update` equivalent can be performed.

## `svn switch url`

    git fetch
    git checkout branch
    git merge

Changing branches should only done without local modification (although it
will simply fail if there are modifications that collide with differences
between the branches).

There is also the complication that branches don't necessarly exist
locally yet. `git checkout branch` will do the proper thing, namely
checking out the corresponding remote branch as a local branch which
tracks the remote one. If it already exist, `git checkout` will simply
switch to that local branch. To actually mimic the svn operation of
getting the current state, we need a pull afterwards. And to make sure
we get the current state when the branch is new locally, we need to
fetch beforehand.
