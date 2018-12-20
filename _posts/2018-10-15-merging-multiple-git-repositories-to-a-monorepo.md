---
layout: post
title: Merging multiple git repositories to a monorepo
canonical: https://medium.com/@MeneDev/merging-multiple-git-repositories-to-a-monorepo-de62d85a2c7a
---

A few weeks ago we started a small project for a new customer in our usual pattern:
several components, each with their own repository and respective CI.
However, it quickly became evident that this introduced orchestration overhead
without any benefit: no versioning, no reuse, just a few components that had
to be bundled into one project for one customer.  
The whole project kept chanting one word: monorepo!

## Goals

 * One of the existing repositories becomes the new monorepo
 * No history rewrite in the main repository
 * Preserve commit history of all repositories
 * Subproject repositories will reside in their own subfolder

## Limitations
 * Order commits by date and interlace commits of formally different repositories would require a few more steps and creating a new repository / completely rewrite the history. That’s just due to the way git works.
 * You may need to fix a few things like your CI afterwards.

## 1 . Prepare repositories that will be merged

As a first step we move every file in each repository that we want to merge to a new subfolder. That’s just a neat trick to prevent merge conflicts. In order to do that you have to make sure there are no uncommitted changes first.

Say we want to call our subfolder “subproject”

```bash
git filter-branch --tree-filter 'mkdir subproject &&
  find . -mindepth 1 -maxdepth 1 ! -name subproject -exec mv {} subproject/ \;' HEAD
```

This will modify each commit in the following fashion:

 * Create a folder named “subproject”
 * List all files in the root directory (-mindepth 1 -maxdepth 1)
 * Ignore the newly created “subproject” folder
 * Move all other files the new “subproject” folder

## 2. Add repository to be merged as remote in monorepo

In the monorepo we need to add the repository of the subproject as new remote. Note that the remote can be a files system location, so we don’t need to push the repository of the subproject (which would require a force push).

```bash
git remote add subproject /full/path/to/subproject-repo
git fetch subproject
```

To conclude this step we fetch the new remote. Double check that you don’t pull a this point.

## 3. Cherry-pick commits from subproject

We could now call git log subproject/master and cherry-pick each commit we want to include in our monorepo. But of course it’s easier and less error-prone to write a script for that ;)

```bash
#!/bin/env bash

if [[ -z $1 ]]; then
    echo "Usage $(basename $0) <remote-name>"
    exit
fi

echo "About to replay commits from remote $1"
read -p "Press enter key to continue, CTRL+C to abort" < /dev/tty

while read sha
do
    subject="$(git log "$sha" --format=format:"%s")"
    echo
    echo "About to cherry-pick $sha"
    echo "$subject"
    echo

    read -n 1 -s -r -p "Press s to skip, any to continue, CTRL+C to abort" key < /dev/tty
    echo
    if [ "$key" == "s" ]; then
        echo "Skipping..."
        continue
    fi
    git cherry-pick "$sha" || exit 1

done <<< "$(git log --reverse $1/master --format=format:"%H")"
```

Save that script outside our monorepo (to avoid uncommitted changes) and make it executable. It expects our remote name as it’s only argument.

Before each commit is cherry-picked we see the SHA of the commit and the first line of the commit message and get to decide what happens:

 * Press `s` to skip
 * `CTRL+C` to stop
 * Any other key to apply the commit

This might be impractical for long commit histories, but in my case it conveniently allowed me to alter a commit using [GitKraken](https://www.gitkraken.com/) before applying the next commit.


## 4. Repeat with other subprojects
Once we’re done with all subprojects we’ll have a new repository that combines all old repositories into one neat monorepo and all commits replayed to preserve their history.