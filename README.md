# How Git Works
-----------------------------------------------------------------------------------------------------
## At its core, Git is a file system:
* Git internally stores these objects in the objects folder (inside .git folder) in a persisted hashtable (dictionary), using the hash of the object as its key and the object as its value
* Hash of a blob or a tree is calculated from SHA1 hash of the content of the object
* Git internally stores a file as a blob object, content of a blob is the compressed form of the file content
* Example blob content printed by: git cat-file -p 9eed377bbdeb4aa5d14f8df9cd50fed042f41023

  Apple Pie
* Git internally stores contents of a folder as a tree object, content of a tree object is the compressed form of the list of hashes, names, read/write/execute permissions of the included files and (sub)trees (subtrees correspond to the subfolders)
* Tree does not contain the name of the folder itself, it contains only the contents (blobs=files and trees=folders) inside this folder
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
