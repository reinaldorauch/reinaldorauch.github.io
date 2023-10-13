---
title: "Git: How to correct missing object (tree,blob,others) errors"
layout: post
comments: true
lang: "en-US"
---


## The problem

<a href="#tldr">go to tldr</a>

So, you're working on a big repo with a lot of changes upstream every day and then you run a `git fetch` command and get a output like this:

![Bad object outputs](/assets/bad-object-sample.png)

If you never came across this `fatal: bad tree object` message you may start panicking and and run around in circles thinking you're doomed and you'll need to download everything again:

<iframe src="https://giphy.com/embed/rg2ielv1QCTeRfJc56" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/disneychannelofficial-disney-channel-disneychannel-amphibia-rg2ielv1QCTeRfJc56">via GIPHY</a></p>

Well, that's one way to do it but for large repositories that may not scale well and you could lose many hours downloading it again:

![Not being able to do your work](https://imgs.xkcd.com/comics/compiling.png)

But there is a one-liner that may fix it more efficiently without needing to download everything again:

## Solution

<h3 id="tldr">tldr</h3>

```bash
git fsck --lost-found | grep missing | awk '{print $3}' | xargs git fetch-pack <the origin repo URI>
```

### Explanation

Here goes: The error ocurrs because some objects (internal git objects: files, diffs, commits, blobs) were lost between your local repo and your upstream (aka: origin) repo. This is something beyond that `git fetch` or `git pull` can fix on it's own so we'll need to wield some not-so-much known git commands to be able to fix them:

```
$ man git fsck

GIT-FSCK(1) Git Manual GIT-FSCK(1)

NAME
       git-fsck - Verifies the connectivity and validity of the objects in the database

SYNOPSIS
       git fsck [--tags] [--root] [--unreachable] [--cache] [--no-reflogs]
                [--[no-]full] [--strict] [--verbose] [--lost-found]
                [--[no-]dangling] [--[no-]progress] [--connectivity-only]
                [--[no-]name-objects] [<object>...]

DESCRIPTION
       Verifies the connectivity and validity of the objects in the database.
```

- `git fsck --lost-found` in a fashion very much like `chkdsk` in windows world and it's linux counterpart `fsck`, it checks the repo database of objects for something wrong with objects, if they're linking with each other correctly and such things (remember git stores only the changes about a file after it is added to the repo, so it needs to know the changes of it across commits to recreate it in the workdir of the repo). In the one-liner, it's function is to return the objects that are missing so we can pull them from the upstream repo.
- `grep missing` is there to filter out the unwanted output from the first command so we only get the lines that have the missing object ids
- `awk '{print $3}'` remove other strings than the object id itself to be able to pass them as arguments to the final command. `awk` is usually installed with your distro but if it's not you can easlly install it from your package manager of choice.
- `xargs git fetch-pack <the origin repo URI>`: Now, here things goes wild: first, `xargs` receives a list of newline-separated (`\n`) strings and appends them as arguments to the following command:

```
$ man git fetch-pack

GIT-FETCH-PACK(1) Git Manual GIT-FETCH-PACK(1)

NAME
       git-fetch-pack - Receive missing objects from another repository

SYNOPSIS
       git fetch-pack [--all] [--quiet|-q] [--keep|-k] [--thin] [--include-tag]
               [--upload-pack=<git-upload-pack>]
               [--depth=<n>] [--no-progress]
               [-v] <repository> [<refs>...]

DESCRIPTION
       Usually you would want to use git fetch, which is a higher level wrapper of this command, instead.

       Invokes git-upload-pack on a possibly remote repository and asks it to send objects missing from this repository, to update the named heads. The list of commits available
       locally is found out by scanning the local refs/ hierarchy and sent to git-upload-pack running on the other end.

       This command degenerates to download everything to complete the asked refs from the remote side when the local side does not have a common ancestor commit.

```

It does the heavy lifting of going to the specified repo URL and asks for the missnig objects that we want to download. You'll need to specify the URL because this is a more low level command that the more-usually-used `git fetch` uses to do it's work. So as for example:

```bash
git fetch-pack git@github.com:reinaldorauch/reinaldorauch.dev.br.git 
```

Will download the entire tree of missing objects from the specified repo to our own. But we will pass as arguments the object ids that we want so it will limit it's search to only objects that lead to them.

And bam! If we run the `git gc` command to make git do the housekeeping stuff in our repo and keep it tidy or run another `git fetch`, them will not report the `fatal: bad tree object` error again because now they will be in your local repo copied from the source repo!

If you got down here, thanks a lot for your time!

You can add comment's down below if you want!

Cheers,

Reinaldo