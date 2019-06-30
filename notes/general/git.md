# Rob's Git Notes

* Git was originally built as a _toolkit_ for creating version control systems.  The polished VCS came later but after Git developed a reputation has having a complicated UI.  Git commands that are part of the polished VCS UI are reffered to as "porcelain" commands while lower-level commands are reffered to as "plumbing" commands.
* Unlike other version control systems, you don't "check-out" files from a git repository.  Instead, you copy the entire repository, inlcuding version history for every file, from the server

## Git Basics

### Common Git Commands
* `git init` - creates a repository in your current directory
* `git clone` - clones a repository into your current directory
* `git add` - begins tracking a file
  * Version of staged file is state of file when `git add` executed.  If file is modified after inclusion in `git add`, must run again to ensure latest is committed.
* `git status` - shows status of files in your branch and how your branch tracks with server
  * '-s' option provides short status.  Output has two columns that track status in staging area and status in working tree.
* `git commit` - commits staged files to repository
  * Text displayed automatically in commit message editor are comments.  These won't be included in saved commit message (unless uncommented).  The diff can be included in default comments with `-v` option on `commit`.
  * Commit message editor can be set with `git config --global core.editor`
  * Commit messages can be written in0line with `-m` option (message in quotes)
  * `-a` option automatically commits any tracked files with changes (essentially skipping the staging step and `git add` commands)
* `git diff` -  shows actual differences between files in working directory, staging area, and committed changes
  * without arguments, compares working directory to staging area
  * To compare staged against committed files, use `git diff --staged`
  * `git difftool` lets you view differences with program of choice like vim
* `git rm` - removes file from staging area.
  * If a file is removed using the system `rm` command, it will show up under "Changes not staged for commit:".  Must run `git rm` to stage the file removal.
  * To delete a staged file, must use the `-f` option to force the removal.  This is a safety feature to ensure uncommitted work isn't lost.
  * To remove a file from staging area but keep on hard drive - for example if you forgot to add it to `.gitignore` - use `git rm --cached`
* `git mv` - renames a file
  * Git actually doesn't track file name history.  `git mv` is exactly equivalent to renaming file and then running `git rm OldName` followed by `git add NewName`.  This means that if you rename a file incidentally, running `git rm` followed by `git add` is totally sufficient.
* `git log` - shows commit history - by default, history of commit messages 
  * `-[number]` - displays only last [number] of commits
  * `-p` - _patch output_ shows the diff introduced by each commit
  * `--stat` shows shows brief commit stats - how many files changed, how many lines, etc.
  * `--pretty` lets you choose one of several different output formats.  `--pretty=format` lets you fully specify the output.
  * `--graph` produces an ASCII graph showing branch and merge history
  * `-S` takes a string and displays only commits that altered the number of occurences of that string
### `.gitignore`
* Contains patterns of files in working branch that Git should ignore
* Comments are begun with `#`
* Standard Glob patterns work:
  * `*` matches zero or more characters
  * `[xyz]` matches x or y or z
  * `?` matches a single character
* [Github has comprehensive list of examples](https://github.com/github/gitignore/)
* `.gitignore` applies recursively to subfolders.  If a subfolder contains its own `gitignore` that masks the higher level one.

### Undoing Things
* If you forget to add files or changes to a commit you've made, make the changes and stage the files, then commit using the `--amend` option.
* If you stage a file and want to unstage it, use `git reset HEAD <filename>`.  These instructions are displayed with every `git status` output.
* If you've modified a file but want to discard the changes and revert two what's in the repo, use `git checkout -- <filename>`.  This too is displayed in `git status` if you have an unstaged file with changes.
 
### Remotes
* Remote repositories are hosted elsewhere and are generally read-only or have write privileges
* `git remote` lists the shortname of each remote handle for the current repository.  If the repository was created by cloning, should at least see `origin` which is the default name of server a repo is cloned from.  `-v` will show the URLs stored for the servers.
* To add a remote repository, use `git remote add <shortname> <url>`.  This will by default add two locations, one for fetching (retrieving remote files) and one for pushing (committing files to remote).  You can get files from a remote repository with `git fetch <shortname>`.   `git fetch origin` downloads all additions to a repository that appeared after you cloned it.
* If a branch is set to track a remote branch, `git pull` will automatically fetch a remote branch and merge into local master.
* To push local master branch to origin server, use `git push origin master`  This will not work if somebody else as pushed changes before you.  You need to pull to merge changes into your local repository before pushing to remote.
* `git remote show <remote>` will display information about a remote, including URL and tracking branch information.
* `git remote rename` changes the name of the name of the remote (local references to a remote repo)
* `git remote rm` removes the remote (local repo unaffected)

## Branching and Merging
* Because breanching in Git is very lightweight (and doesn't require a wholesale copy), branching can be used generously to develop modfifications of a main branch, test, and merge changes.  For example, working on a single bug could be an excuse to create a branch solely to write and test a bugfix before merging into a master main branch and pushing to production.  These short-lived branches are called topic branches.
* `git branch` is used to create a new branch and `git checkout` will switch to that branch.  `git checkout -b` will create a new branch and switch to it.
* As described in the section below, a commit creates an object that points to a snapshot of the repo.  Subsequent commits point to previous commits which creates a history of the repo.  
* When you move between branches using `git commit`, what actually changes is the value of a pointer called `HEAD` which describes the bracnch being pointed to.  The branch itself points at a commit object representing the current state of the repo (for that branch).
* If you have unstaged changed (that aren't committed to the branch you're working on), you must commit (or use a mechanism called stashing) before switching branches
* Once you're ready to integrate working branch into master branch, use `git merge`.  Assuming no other updates have happened to the main branch since the initial branching, the merge is called a "fast-forward".  This is possible when merging a commit with another commit that can be reached by following the first's commit history.  In this case, the current state of the master branch is a parent state of the newer branch.  Under the hood, the master branch's pointer is just set to the new branch's commit object.  After merging, the branch still exists and can be deleted with `git branch -d`.
* When merging a branch that isn't a direct descendent, git will automatically try to merge the two.  If the same part of a file was edited, this will fail due to the conflict.  After a failed merge, `git status` will show the conflicts.  The file with conflicts will have those conflicts highlighted.  To resolve, delete the annotations and make the file how you want it.  Then run `git add` and commit to resolve.  `git mergetool` will launch an graphical resolution tool.
* `git branch --merged` shows branches that have been merged into current branch.  Can safely delete these (except current branch which is listed) with `git branch -d`.  `git branch --no-merged` shows branches with unmerged changes.

## Remotes
* `git fetch` will update your local machine with the state of a remote repository.  This creates a _remote-tracking_ branch locally with the name [remote-name]/[branch-name].  Remote-tracking branches aren't editable branches - they simply represent the current state of the remote branch.  To merge remote-tracking branches into an editable branch on your machine, use `git merge <remote-tracking name>`.  To create an editable local branch thats initialized from a remote-tracking branch, use `git checkout -b <new branch name> <remote-tracking branch>`.  You can shortcut this with `git checkout <remote-branch name>` if the branch exists on only one remote and doesn't exist locally.
*  Initializing a local branch from a remote branch creates a "tracking branch" where the remote branch is the "upstream" branch.  Commits made to the local tracking branch are said to be "ahead" of the remote branch.  Commits made to the remote branch not merged into local are said to be "behind".  `git status -vv` shows the state of all your local branches, what branches they're tracking, and how many commmits ahead or behind they are.  When you clone from a remote, a tracking branch is automatically created to track the origin.  You can create a remote branch, and set it to be the upstream branch, with `git branch -u`.
* Pushing to a remote doesn't have to be just for merging with a remote master branch.  If you have a local branch that you want to make collaborative with other team members, use `git push <remote> <branch>` to push the local branch to a remote.
* `git fetch` is used to actually reach out on network and get updated status of remotes.
* `git pull` is a shortcut for `git fetch` followed by `git merge`.

## Git Objects
* Git is fundamentally a content-addressed filesystem with a version control system built on top of it.  A content-addressed filesystem is basically a key-value system.  Data is stored and then hashed to produce a key.  This key is handed back and must be used for future retrieval.  This is in contrast to a location-addressed file system where, after storing content, the storage location is used for future retrieval.  A big difference is location-based methods are agnostic to the data contents.
* Unlike some other version control systems, Git stores *snapshots* of the versions of files - instead of just the differences.  The _entire_ file is compressed and stored.
* The plumbing command `git hash-object` takes data and hashes the content to create the key that would be used to store it.  The `-w` option will actually store it in the _objects database_ located in `.git/objects`.  SHA-1 hash is used to produce the key.

    ```
    $ echo 'test content' | git hash-object -w --stdin
    d670460b4b4aece5915caf5c68d12f560a9fe3e4
    ```

* The `.git/objects` directory contains the contents of the stored files compressed into "blobs".  The subdirectories and files in those subdirectories are named the hashes of the original contents (the value returned by `hash-object`).  The folder containing the blob is named after the first two characters of the hash.  The rest of the hash is the blob file name.  In the above example, the blob of 'test content' would be stored in `.git/objects/d6/704...`   
* `git cat-file` will take a key and can display the content.
* This _content addresssed filesystem_ plays well with a version control system because stores and retains information based on the content.  Assuming I've already hashed, blobbed, and stored "program.cc", the moment I edit "program.cc", storing it again will result in a different key.  Both the orignial and new versions will be saved (and could be retrieved)
* So far, this system has required retaining the hash values to retrieve data.  Obviously this is impractical.
* In addition to blobs which are file contents, the git object directory contains _trees_ which represent directories in the repository.  Each tree object lists the files (blobs) or subdirectories (trees)  within it.  Each entry consists of:
  * The actual file or directory name
  * The content hash that would be used to retrieve it
  * type (blob or tree)
  * A file mode that can determine whether file is normal file, executable file, etc.
* Modifications to the tree+blob file system are staged before execution.  This staging area is called an index.  The plumbing commands `update-index` and `write-tree` are used to generate indicies and trees from those indicies respectively.
* Trees along with blobs enable encoding a set directory structure and fixed file contents and make them uniquely accessible from the top level tree hash value.  This structure is used to store the state of a file repository hashed to a single value.  Commit objects track these snapshots by storing the hash of the top level tree, the author/committer, and the commit message.


## Ignoring Files
In addition to `.gitignore` files in the repository, can add additional ignore patterns to `{repo root}/.git/info/exclude`.
