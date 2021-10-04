# How Git Works
-----------------------------------------------------------------------------------------------------
## At its core, Git is a file system:
* Git internally stores a file as a blob object, content of a blob is the compressed form of the file content
* Example blob content printed by: git cat-file -p 9eed377bbdeb4aa5d14f8df9cd50fed042f41023

  Apple Pie
* Git internally stores contents of a folder as a tree object, content of a tree object is the compressed form of the list of hashes, names, read/write/execute permissions of the included files and (sub)trees (subtrees correspond to the subfolders)
* Tree does not contain the name of the folder itself, it contains only the contents (blobs=files and trees=folders) inside this folder
* Git's internal objects folder (inside .git folder) is organized as a persisted hashtable (dictionary), using the hash of the object as its key and the object as its value
* Hash of a blob or a tree is calculated from SHA1 hash of the content of the object
* Example tree content printed by: git cat-file -p 46624f2d142b3a74e3aac65f691cdfdbb42ce022 (contents of "cookbook" folder)

  100644 blob 9eed377bbdeb4aa5d14f8df9cd50fed042f41023    menu.txt
  
  040000 tree 5458b1e1046e66dd8dbb249fc4ff64ab29ff90a9    recipes
* Example tree content printed by: git cat-file -p 5458b1e1046e66dd8dbb249fc4ff64ab29ff90a9 (contents of "recipes" folder)

  100644 blob 37c0180c056cc7bf91b1046c4990ec5ee71668f2    README.txt
 
  100644 blob 9eed377bbdeb4aa5d14f8df9cd50fed042f41023    apple_pie.txt
* In these examples, menu.txt and apple_pie.txt, even though they are in different folders, have the same content, same hash, so they are internally stored in a single location
* If a file has the same content as another file, their hashes are the same, so its blob is not duplicated in internal git storage
* However, if any file has a different content or name or read/write/execute permissions, the tree containing these files will have different content and a different hash, so the tree will be internally stored in a different hash location
* Hash references to the same file or tree work very similar to how symbolic links in linux (or shortcuts in windows) work, they are plain references
* Git objects which are internally stored in the objects folder (inside .git folder) are immutable, their contents do not change
-----------------------------------------------------------------------------------------------------
## At the outer layer, Git is a change tracker:
* Git internally stores a commit in the objects folder (inside .git folder) as a commit object, content of which is the hash of a new tree, author info, commit message, and the hash of the previous commit on this branch (the parent commit object), a merge-commit has two parents
* Example commit content printed by: git cat-file -p fbae55d7f2a7e578d24054d631f7a793fe2a1116

  tree 46624f2d142b3a74e3aac65f691cdfdbb42ce022
 
  author koda <kdakan@gmail.com> 1544779606 +0300
 
  committer koda <kdakan@gmail.com> 1544779606 +0300
 
  First commit!
* Example commit content printed by: git cat-file -p 725750d59477f3186eda11ce20b9fba4b1cdc45f

  tree 8a913a85d035f9b3f4097087520dec7cfbada5b0
 
  parent fbae55d7f2a7e578d24054d631f7a793fe2a1116
 
  author koda <kdakan@gmail.com> 1544781927 +0300
 
  committer koda <kdakan@gmail.com> 1544781927 +0300
 
  Add cake
* Git internally stores an annotated tag in the tags folder (inside .git\refs folder) as the hash of the latest commit, tag annotation, tag message and tagger info
* Example tag content printed by: git cat-file -p mytag

  object 725750d59477f3186eda11ce20b9fba4b1cdc45f
 
  type commit
 
  tag mytag
 
  tagger koda <kdakan@gmail.com> 1544783841 +0300
 
  I love cheesecake
-----------------------------------------------------------------------------------------------------
## Branching, merging, fast-forward merge, rebasing, and experimenting code in detached-head:
* Git internally stores a branch in the heads folder (inside .git\refs folder), initially as the hash of the latest commit
* After each commit, Git updates the branch reference with the hash of the new commit
* When a Git repo is first initialized, a branch named "master" is automatically created with empty content (because there is no commit yet)
* Git internally stores the current branch name in the HEAD file (inside .git folder)
* When switching to another branch, Git updates the HEAD file (inside .git folder) with the name of the new current branch and replaces the contents of the working folder with the contents referenced by the branch's commit object fetched from the internal Git storage
* Git internally stores a merge-commit same way it does the non-merge commit, but instead with two parent commit object hashes
* However, when the already merged branch is merged with the other branch, Git does not create a new commit, instead it updates the branch in the heads folder (inside .git\refs folder) with the hash of the latest commit on the other branch (this is called fast-forward)
* There is also a Head-detached operation which works without any branch by checking out directly a commit for experimenting with code, and in this mode, commits can later be dismissed by garbage collection operation, or attached to a new branch if needed, but this is normally not used
* Rebase operation is like a merge, but deletes the history of the merged branch and adds copies of its commits on top (at the end of) of the destination branch's commits, to help simplify tracking the commit history when there too many parallel branches
* Rebase operation changes the ordering of the merged commits, and can sometimes cause trouble, prefer merge instead of rebase in general
* A tag is similar to a branch, except that branches move but tags stay constant
-----------------------------------------------------------------------------------------------------
## At the outermost layer, Git is distributed:
* Cloning copies the .git folder from a remote repository branch (master branch by default) to the folder in the local machine, and recreates the working folder from this internal storage
* Cloned repositories are exactly the same, and same as the remote origin repository
* Remote branches are stored internally in the packed-refs folder (inside .git folder), git show-ref shows both local and remote branches
* Pushing updates the remote origin branch commit hash and the info stored in the packed-refs folder (inside .git folder)
* If there is another commit pushed by some other peer, and it conflicts with our local commit, we cannot push our commit (unless push by force, but this will delete the other commit and not recommended)
* In this case, we should fetch and merge (resolve conflicts locally), then push again, fetch and merge together is done by the git pull command
* Pushing after rebasing can cause problems because rebase creates copies of commits from the source branch and deletes the old commits on that branch, but there may be other commits on that same branch in the remote repository, so do not use rebase when working on a distributed repo
-----------------------------------------------------------------------------------------------------
## Working with a Git repository hosted on Github:
* You do not have permission to clone and push someone else's repository, so you "fork" it into your own Github repository
* Then you can clone your own Github repository and push your commits there
* If you want your commits to go into the original author's repository, you first define another origin for that repository (called "upstream"), and then you send him a "pull request"
* "fork" and "pull request" are not features of Git itself, they are features offered by Github
-----------------------------------------------------------------------------------------------------
## Git commands for configuring a local repo:
* git config --global user.name "koda" (set global user name)
* git config --global user.email kdakan@gmail.com (set global user email)
* git config --global color.ui true (use some colors on the command line)
* git config --global user.name (shows global user name)
* git init (initialize a new repo in the working folder, creates the hidden .git folder for internal storage)
* git config user.name "koda" (set local user name for this repo)
* git config user.name (shows local user name for this repo)
* git config --global alias.st "status" (defines a global alias for the command "git status" as "git st", used mostly to shorten or simplify routinely used commands)
* git config --global core.autocrlf true (changes lf character to crlf characters when working with a file and back into a lf character when committing the file, use this when you're working on a Windows system and other people are contributing to the same repo using linux/unix/macos)
* git config --global core.autocrlf input (changes crlf character to lf character when working with a file, use this when you're working on linux/unix/macos)
* git clone myRepo backupOfMyRepo (clones myRepo into another repo named backupOfMyRepo)
-----------------------------------------------------------------------------------------------------
## Git commands for working within a local branch:
* git status (shows the current branch and status of the files tracked and untracked by Git inside the working folder, all files either staged, or unstaged, or already committed, are tracked for changes by their modified timestamp and size, and all changed files show up with the git status command)
* git add xyz.txt (adds xyz.txt into the "staging area")
* git add . and git add --all (stage all unstaged files and folders)
* git rm --cached xyz.txt (removes xyz.txt from the "staging area" but does not delete the file on the working folder)
* git rm xyz.txt (removes xyz.txt from the "staging area" and deletes the file on the working folder)
* git reset HEAD xyz.txt (also does the same thing, unstages this file) 
* git reset HEAD (unstages all files in the "staging area")
* git commit -m "Description about this commit" (creates a commit with the files and folders in the "staging area")
* git checkout -- xyz.txt (restores the file in the working folder with the committed version)
* git log (lists the commit history for the current branch)
* git log -all (lists the commit history for all branches)
* git log --oneline (lists the commit history for the current branch, each commit on one line)
* git log --oneline patch (lists the commit history for the current branch, each commit on one line and then the changes in the files in that commit)
* git log --oneline graph (lists the commit history for the current branch, each commit on one line, including the visual graph of branching and merging points)
* git log --since=2018-01-01 --until=2018-01-31 (lists the commit history for the current branch, from dates 2018-01-01 to 2018-01-31)
* git diff (shows differences in unstaged files from their committed version)
* git diff --staged (shows differences in staged files from their committed version)
* git diff 548xdfcew8fc..fd78dfg8cds8g (shows differences in files changed from commit 548xdfcew8fc to commit fd78dfg8cds8g)
* git diff master mybranch (shows differences in files from different branches)
* git blame xyz.txt --date short (shows commits which changed xyz.txt)
* git reset --soft HEAD^ (deletes the last commit, so that committed files are added back to the "staging area")
* git reset --hard HEAD^ (deletes the last commit without adding committed files back to the "staging area", so all changes are gone and git status shows nothing)
* git reset --hard HEAD^^ (deletes the last two commits without adding committed files back to the "staging area")
* git reset --hard sd78976sd3 (undoes the last commits back to and excluding commit sd78976sd3)
* git reflog (shows all commits, including the commits deleted by git reset --hard and commits from deleted branches)
* git log --walk-ref-log (shows detailed info of all commits, including the commits deleted by git reset --hard and commits from deleted branches)
* git reset --hard 34r3dscsr7 (moves HEAD back to a previously deleted latest commit 34r3dscsr7, so that this commit is recovered)
* git commit --amend (adds files in the "staging area" to the last commit)
* git commit --amend -m "Updated commit description" (adds files in the "staging area" to the last commit and updates the last commit message)
* git reset --... and git commit --amend commands should not be used after pushing the commit to a remote repo (changing history is a bad thing to do on remote repos)
* git tag -a v0.1.2 -m "Version 0.1.2" (creates a new local tag, tags are mostly used for release versioning)
* git tag (shows all local tags)
* git checkout v0.1.2 (checks out the commit for this tag)
* git push --tags (pushes all local tags to remote repo)
* git rebase -i HEAD~3 (allows interactively to change the commit history for last 3 commits, like editing the commit messages or squashing/melding commits into one or reordering commits)
* git filter-branch --tree-filter "rm -f password.txt" (removes accidentally committed password.txt from commit history for the current branch)
* git filter-branch --tree-filter "rm -f password.txt" -- --all (removes accidentally committed password.txt from commit history for all branches)
* git filter-branch -f --prune-empty -- --all (removes empty commits from commit history for all branches)
* git stash save or git stash (saves modified files to a new temporary area and restores files in the working folder from last commit) (Git adds a new stash, so there may be multiple stashes at the same time)
* git stash save --keep-index (stashes only the unstaged modified files)
* git stash save --include-untracked (also stashes untracked files)
* git stash apply or git stash apply stash@{0}(restores the latest stashed files from the temporary area) (Git does not remove the stash)
* git stash apply stash@{1}(restores the latest stashed files from the latest second stash temporary area (stash{0} means latest stash)
* git stash drop or git stash drop stash@ (removes the last stash)
* git stash pop (same as git stash apply and then git stash drop)
* git stash show or git stash show stash@{0} (shows the last stash)
* git stash list (shows all stashes and the last commit before each stash)
* git stash list --stat (shows all stashes and the last commit before each stash, as well as the file names)
* git stash clear (removes all stashes)
-----------------------------------------------------------------------------------------------------
## Git commands for working with branches:
* git branch mybranch (creates a branch named mybranch from the current branch)
* git branch (lists all local branch names)
* git branch -r (lists all remote branch names)
* git checkout mybranch (checks out the branch named mybranch, recreating the working folder contents from this branch) (it does not matter whether it is a local branch or a remote branch)
* git checkout -b mybranch (creates a branch named mybranch from the current branch and checks it out)
* git merge mybranch (merges the branch named mybranch onto the current branch) (if it is a fast-forward merge where files did not change, Git does not create a new commit but just moves HEAD forward, else Git creates a new commit with the changed files)
* git branch -d mybranch (deletes the local branch named mybranch, does not work if there are unmerged commits on this branch)
* git branch -D mybranch (deletes the local branch named mybranch, even if there are unmerged commits on this branch)
* git branch mybranch fgdsg87egsd63 (re-creates the previously deleted branch from commit fgdsg87egsd63)
* git push origin :mybranch (deletes the remote branch named mybranch)
* git remote prune origin (removel stale local branches that were deleted on the remote repo)
* git rebase master (rebases the current branch on top of the master branch, meaning Git updates the starting point of the current branch with the latest commit on the master branch and then adds/reapplies commits of the current branch from there, as if the current branch with all its commits were created new) (do not rebase someone else's branch) 
* git cherry-pick 43sd874fds349 (picks and copies a commit from a different branch and adds it onto the current branch)
-----------------------------------------------------------------------------------------------------
## Git commands for working with a remote repo:
* git remote add origin https://github.com/kdakan/the-remote-repo (adds a remote repo with the name "origin" and the given url, the remote name "origin" can be anything but used conventionally)
* git remote -v (shows all the remote repos for this repo)
* git remote rm origin (removes the remote repo with the name "origin")
* git remote show origin (shows info about the remote repo and the remote branches on that repo)
* git push -u origin master (pushes the local master branch to the remote repo named origin) (asks for user and password for the remote git host, can be cached for later pushes and pulls)
* git push origin mybranch (pushes the local mybranch branch to the remote repo named origin)
* git push (pushes the latest pushed branch to the lates pushed remote repo, so that when you're working on the same branch and same remote repo, you do not have to repeat this information each time)
* git push staging mybranch:master (pushes local branch named mybranch to the remote repo named staging as the master remote branch)
* git pull (fetches changes from the remote repo and merges into the local branches using git merge origin/master)
* git fetch (fetches changes from the remote repo without merging into the local branch)
* git clone https://github.com/kdakan/the-remote-repo (downloads the remote repo .git folder inside the local working folder, recreates local working folder contents, and adds the remote repo as remote origin, and checks out the initial master branch)
* git clone https://github.com/kdakan/the-remote-repo myLocalWorkingFolder (does the same thing but inside the local working folder named myLocalWorkingFolder)
