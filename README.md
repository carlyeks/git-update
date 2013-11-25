# git-update

Tool to update git repositories at intervals

## Configuration

### ~/.update.repos

This lists the repos that are to be updated. Each line lists a repo, a tab, then the frequency to update.
  
```
~/git-update	1w
```

## Usage

`git-update` will update the repositories if it has been at least the required time. `git-update` can be run at each login, or on a cron script.