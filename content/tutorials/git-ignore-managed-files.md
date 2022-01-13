+++
Tags = ["Tutorial","git"]
Description = "Ignore tracked/managed files in git"
date = "2022-01-13T10:00:00+01:00"
title = "Git: ignore tracked files"
Categories = ["Tutorial","Git"]

+++

# Ignore already tracked files in git

When using git, sometimes a file is added by mistake or for demo purpose, which may be removed or not tracked later on if changed.

Normally we just add an entry in **.gitignore**, but if the file is already managed, it still will be tracked.


## Remove a managed file

If we fully want to remove the file from being managed, we can remove it from the cache. After this, the file still exists locally but will be removed from git:

```bash
$ git rm --cache FILENAME
```

## Ignore/Untrack a managed file: update-index

### Igore local changes

To just ignore local changes on a file, you can run the following command.
After this, you can change the file but it will not be recognized as changed and it is also no more managed by git. Meaning a git pull will never change that file even if someone pushed a change.

```bash
$ git update-index --assume-unchanged FILENAME
```

### Exclude from tracking

If there is a file, which has to stay in git, but may change locally and should not be updated, the file can be flagged to be skipped.
After this, you can change the file but it will not be recognized as changed. The file actually is managed by git and therefore will be updated if it's pushed by someone.

```bash
$ git update-index --skip-worktree FILENAME
```

#### Update an excluded file

If a file is ignored by **.gitignore** and excluded from tracking, you even can update it by forcing it:

```
$ git add -f FILENAME
```


