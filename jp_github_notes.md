# JP Github Notes

Here is the [best online open source book on Git](https://git-scm.com/book).  
Here is the [best very short starting guide](https://guides.github.com/activities/hello-world/) that explains the flow of Github very well.  

### The General Flow and Use of Git and Github
The sections below will have more detailed information about what is mentioned here, but I want to give a quick, "plain English" overview.  We use Github to test different approaches to development without interfering with the original (usually functioning) copy.  We do this by creating multiple, isolated silos -- think of them as separated sandboxes on a playground or beach -- called _branches_.  In each branch we can store a version, however incomplete or radical, of the files we are working on.  We can have as many branches as we want, named whatever we want.  Usually a descriptive name about the versions/approaches used in the files stored in it is wise.  

For example, if we have a real estate application we're working on and we want to try two new approaches to predicting what the value of certain locations will be in six months, one approach using the change in population in the areas and the other approach using the amount of new development growth in the areas, then we could use Github to help us.  We would make a new branch for the population approach and one for the new growth.  We would then create/use a single initial file (project, etc.), make the changes for each approach (at this point the files/projects become separate from each other, with unique code and supporting files as they pertain to the specific approach), and then _push_ the given approach's files and docs up to its respective branch.  

While we are doing this, our original project, which works but doesn't have either of the new approaches in it, is safe on the "master" branch.  We can then compare the differences in code between the master branch and the two sub-branches to see if we want to _merge_ the changes from either of the new-approach branches.  Using branches in this way keeps our various new ideas in safe sandboxes until they are deemed ready to be integrated (merged) into the original, existing project codebase.  

It also means that we can have a single file on our local machine and depending on which branch we are actively "using" (or, "in"), we will have access to the versions of those files stored in that branch.  So, we don't need to create two separate files called "`real_estate_prediction_via_population_trial.py`" and "`real_estate_prediction_via_growth_trial.py`".  Instead, we would just have our original file, such as "`real_estate_prediction.py`", and once we entered a specific branch we would upload this file to it, continue to make approach-specific changes, and upload it again after making more changes.  

This version is stored in that branch and that branch only.  If you switch back to the "population" branch from the "growth" branch, then the local `real_estate_prediction.py` file will revert back to the "population" version while leaving the modified "growth" version in the growth branch.  Branches live on your local machine.  Once you sync to Github.com there is a backup of your work online.  But it is still a local system that you can direct and redirect to any repository online.  

And since Github tracks each of your "uploads" (called _commits_) to a given branch, if you screw something up majorly you can just "rewind" the branch to a previous state and use the un-screwed-up versions of the files at that point.  You don't have to keep a massive library of `version1.py`, `version2.py`, etc...., on your local drive.  How could you even tell the differences between all those files?  What a nightmare.  Github and its _diffs_ tracking system are designed to handle just this type of scenario.  Really, really great.

As you can imagine, the concept of using Github is most beneficial to teams of people working on a project together.  Perhaps one team would be assigned to do the "population" approach and test it out; another given the "growth" method and must test it.  They can work separately and when done can compared all their code through Github's _diffs_ and decide which approach they want to merge into the previous "master copy" of their project.  Maybe some code from both?  Who knows?







<BR>

### Commands to use Git and Github

#### Create a New Repository
1. Go to Github and create a new repo.  Copy the repo's url with `.git` at the end (e.g. `github.com/<username>/<repo name>.git`).
2. Navigate to local directory in terminal that you want the new repo's folder to exist in.
3. `git init`
4. `git remote add origin <URL>` adds the origin (the github repo) as the destination for our local folder (paste URL)
5. `git remote -v` to verify the URLs for the repo are correct


#### Create Repo From Existing Local Directory
1. Go to github, create a new repo, name it, copy `.git` URL
    + Note: create this repo _without_ a README, gitignore, or license to avoid errors.  Add these after first commit.
2. Navigate to local directory in terminal
3. `git init`
4. `git remote add origin <URL>` adds the origin (the github repo) as the destination for our local folder (paste URL)
5. `git remote -v` to verify the URLs for the repo are correct
6. `git add .` to add _all_ files in this folder
7. `git commit -m "first upload"` to commit those files to the git head
8. `git push origin master` to push all committed files up to the actual repo. yay!  

__Note 1:__ if you do create the repo on Github with a readme or license in it, or you are trying to sync a new local folder with an existing github repo, you will have to first download all the stuff (e.g. README, license, other files) from the repo to your local folder before you can push any new files from your local back up to your repo.  This merging keeps the two locations, local and repo, in sync.

__Note 2:__ `git remote set-url origin <URL>` is a similar command to \#4, `git remote add origin <URL>`, but it is used once a local folder has already had its origin added and you now need to change it.  You just set the URL to the new desired URL.  You can also simply add a new remote to this branch using `git remote add <shortname> <URL>`


#### Clone an Existing Online Repo
1. Find master repo on Github.
2. Fork to your user profile (this is always the case if you are going to contribute any code to the repo; see below for the "observing repo only" exception).
3. Clone to your local by copying the __clone__ (green button) link *on your Github profile's fork*.
4. `git clone <link>` in terminal in the desired directory you want to download the repo to on your local.

Note it is possible to simply clone a repo from a master, like zipfian, without forking to your own profile.  Do this if that repo is going to be updated regularly by the creator, such as a assignment solutions repo which will have new solutions added daily or weekly.  Simply clone to your desired directory and then update your local copy once changes are made in the zipfian master by using `git pull` in the directory of the cloned repo.  

This pull request will simply match your local files to the zipfian master, which is all we want since we won't be branching anything in a simple solutions repo.  Also, if you have _added_ files to your local, this will either attempt to upload them to git or cause an error as the two are not synced (you'll have to delete/move them if you want to `pull` from the master repo unless the master repo agrees to be updated by having those files added to it permanently).


#### Pushing Modified Files to a Repo
1. ...Make changes to files / do work...
2. `git add <filename>` - in <filename> directory in terminal to add changes to your staging area (`git add .` adds _all_ files).
3. `git commit -m, "<change summary>"` - to commit the changed files with summary of changes (this is required; commit will be rejected without).
4. `git push origin master` - to push committed files back to your repo.  `origin` refers to the repo from which you cloned originally; in this case, your personal repo.  `master` refers to the branch in that repo you are pushing to. If you want to push to a different branch, use its name here.  Same for a different repo.  


#### Making a Pull Request to Have Your Work Merged with an Upstream Repo
1. Create a pull request (PR) for the upstream target branch (e.g. your boss's main branch) by going to the target repo on Github.com (by default that you forked from in), and choosing '_create new pull request_' and following the steps there.  
    This is asking the upstream/target branch to "pull" the changes you are pushing into their repo, overwriting their files.  This will only be done once code checks and testing have been done to ensure that such a merge won't ruin the functionality of the existing codebase (i.e. Pull requests are sort of the 'final step' in contributing code).
2. Once a code review has been done, the changes in the Pull Request have to be _merged_ into `master`.  Do this on Github.com.

#### Creating, Changing, and Using Branches
Branches allow you to upload (push) any changes to given files to an isolated "sandbox", if you will.  Branches are created on your local machine first.  They serve as siloed storage spaces for any files.  For example, you can have three branches, named `testing-1`, `testing-2`, `testing-3`, and can push three different versions of the same files to each.  Say one version is using old code you tried to modify and another version is using all new code from scratch.  You don't want to have two different copies of the same file in your local directory, and you don't want to have to be locked into only having one approach.  So, you edit the files as needed and push each "version" or "type" of approach to a separate branch.  From there you can then submit a pull request to a master branch from any/all of these branches.  The pull request will show the differences (diffs) between each branch and the master branch.  
1. `git branch <name>` - creates a new branch (on your local, technically) with <name>
2. `git checkout <branch name>` - switches to a specified branch.  `git checkout testing-1` puts you on `testing-1`
3. `git push origin <branch>` - push changes as usual (can push to anyone's branch in the repo, if desired.  Note again `origin` means basically _your_ repo, but can push to anyone else's branch in _their_ repo if you set that target as a remote for your branch).
4. Delete a no-longer-needed branch from the __Branches__ tab in the repo.

We can use `git fetch <remote-name>` to simply fetch all the data in that remote that we don't have.  It only downloads this data to our local, it does not merge it with any of our work or modify what we're currently working on.  That will have to be done manually in this case.

<BR>

###### Light Branch Troubleshooting
If there is a merge conflict...
Switch into MASTER branch and `git pull` to make sure you've got all the updated files from the master branch.  Anytime you go into a new branch, just `pull` the branch to grab its files.
THEN go back into YOUR branch and `git merge` into the master, once you've got files that you want to merge.

It is possible to simply keep working in your branch for a while, and merge after much work has been done, but never ever `merge` into master branch without doing a `pull` first.  Always, always `pull` from MASTER before merging into it.  Otherwise you get big merge conflicts because files in different branches are not matching up.  Bad.


<BR>

#### Pulling Modified Files From a Repo
`git pull <target repo> <target branch>` to sync your branch with the target branch (alters your local files)



#### Merging my Fork with the Original Upstream Repo
Once in the local repo *of my fork* in terminal:
`git remote add upstream [url for the upstream original repo I am syncing from]`
`git pull upstream master`



#### The `.gitignore` File
in the main level folder of each repo is a `.gitignore` file.
In it, list the files or file types in that folder that you want github to ignore when syncing with your repo.  For example, if you have a big database or a file with your passwords in it, you would list them on a new line each, such as: `my_database.csv` or `my_passwords.json`  and they would be ignored, not uploaded to github.
If you wish to simply ignore all occurrences of a file type, you simply prefix the file type itself with an asterisk, such as: `*.csv` or `*.json` to ignore all csv or json files in that repo.
If you want to ignore a specific folder, simply enter the directory into the `.gitignore` file.  If your directory structure was *Top_Level* -> *Data_A*, *Data_B* -> *Script_A*, *Script_B* and your master `.git` folder is in *Top_Level*, you can ignore all occurrences of folder named *Data_B* and its own subfolders by saving `Data_B/` into the `.gitignore` file.  If you want to ignore just that folder but not its subfolders (I think...) you can just enter that folder into the `.gitignore` file, *Data_B*.

Please note that if you delete any file or folder on the Github web interface, it will be deleted from your local once you do a `git pull`.  Your local repo and the corresponding master branch on github are always going to mirror each other.  Hence, branching in git.


[Here is a good reference](https://www.atlassian.com/git/tutorials/gitignore#git-ignore-patterns) on how to glob/pattern match in `.gitignore` files.


#### Set Username and Email
`git config --global user.name` -- your real name  
`git config --global user.email` -- your email



#### Undoing or Fixing Previous Commits
`git commit --amend` -- used for adding some new files or changes into a previous commit that wasn't pushed.  Will overwrite previous commit so it all appears as only a single commit.







#### Github Commands
Command | Action
--------|--------
`git` | displays common git commands and a brief overview.
`git config` | displays configuration commands for your account.
`git init` | initializes a git repository with git commands. This also creates a .git folder within the git repository; *must have this folder to work.*
`git add [file]` | add named file to the git branch/directory and track it. *Note:* `git add .` adds all changes in current  dir.
`git commit -m "type your summary here"` | sets a checkpoint for file/branch versioning in your git. *Note:* must add summary message when committing, use single quotes, keep brief
`git status` | gives status of files, branches, and commits
`git remote -v` | gives target URLs for push/pull in a given folder (repo)
`git remote set-url` | changes URL for push/pull in a given folder (follow instr)
`git log` | gives log of commits. Use `-p` to see diffs and `-2` to limit them to two. `--stat` for summary stats.
`git clone [url]` | clones a git repository into working dir. Do **not** make a new git repository within an existing git repository!  *Note:* usually done to clone a git repo from github to your local drive.  By default you can't edit a repo you cloned from someone else (can edit your own).  Use this to get/create/initialize an copy of an existing online git repository to begin working.  Once cloned, use push/pull to send/receive modified files to/from the online repository.
`git push` | pushes edited git repository & files up to the original github repository. This is how you share changes over github
`git pull` | pulls updated changes from online repository on github down to your computer,	assuming you have that existing repo on your laptop  
`git rm [file]` | removes file/folder *from local machine*. Use `-r` for folders. See: https://git-scm.com/docs/git-rm
`git rm [file] --cached` | removes file/folder from git index but *saves local*.  Once deleted, you must `add`, `commit`, and `push` to your repo to sync it with your local index, to reflect the deletions on your github repo.

+ Some more advanced commands can be found at https://services.github.com/kit/downloads/github-git-cheat-sheet.pdf

<br>

### Git & Github Terminal Glossary

Command | Function
------- | --------
**Repository / Repo** | This is the project's source code that resides on github.com's servers. You cannot modify the contents of this repository directly unless you were the one who created it in the first place.
**Fork** | Forking a project will create a copy of the original repository that you can modify as you please. Forked projects will appear in your own github.com account.
**Cloning** | this will clone an online repository to your hard drive so you may begin working on your modifications. This local copy is called your local repository.
**Branch** | A branch is a different version of the same project. In the case of T2DMIT, you will see 2 branches : the master branch and the development branch.
**Remote** | A remote is simply an alias pointing to an online repository. It is much easier to work with such aliases than typing in the complete URL of online repositories every single time.  Example: `git remote set-url origin <new url>` will replace the old url with the new one as the origin (master) for that repo.  This is very handy.
**Staging Area** | Whenever you want to update your online repository (the one appearing in your github.com account), you first need to add your changes to your staging area. Modifying files locally will not automatically update your staging area's contents.
**Fetch** | git fetch will download the current state (containing updated and newly created branches) of an online repository without modifying your local repository. It places its results in .git/FETCH_HEAD.
**Merge** | git merge will merge the modifications of another branch into the current working branch.
**Pull** | git pull is actually a combination of git fetch and git merge. It fetches the information from an online repository's branch and merges it with your local copy.
**Add** | Whenever you modify a file in your local repository or create a new file, that file will appear as unstaged. Calling git add allows you to specify files to be added to your staging area.
**Commit** | A commit records a snapshot of your staging area, making it ready to be pushed to an online repository.
**Push** | git push will take all of your locally committed changes and upload them to a remote repository's branch.



#### Git Notes:
+ This is the best basic tutorial I've found:
  + https://github.com/GarageGames/Torque2D/wiki/Cloning-the-repo-and-working-with-Git  

+ Visual Cheatsheet:
  + http://ndpsoftware.com/git-cheatsheet.html#loc=remote_repo;

+ Other basic tutorials:
  + https://www.atlassian.com/git/tutorials/
  + http://rogerdudler.github.io/git-guide/

+ Do **not** `commit` very large files, like anything over ~20 MB!

+ When forking, always save forked repo into your account (*jp-wright*) unless otherwise noted.

+ Once you clone a repo (from a forked repo in your online profile), you can move it anywhere on your local HDD and it will be fine.  Git doesn't care about the absolute path on your local HDD, it only wants the relative paths within the cloned repo.  There is a hidden *.git* folder in the top-most level of every repo that has all the important syncing commands.  As long as it is in the repo, your git will work.

+ If we've made changes to the repository after you forked it and you want to update your repository to reflect them, you can use the 'master' command.  For example: `git pull https://github.com/zipfian/precourse master`  (`git fetch` will simply grab the new files if needed)

+ If you try to push and get "Repository does not exist" this probably means that you cloned from the Zipfian repo rather than your fork. Make sure you created a fork to your account. Then run this command to push: `git push https://github.com/<your username>/precourse master`

+ If VIM loads in terminal in response to some commands and prevents you from exiting/acting, hit **shift + colon**, then **w**, then **q**, then **enter** to exit VIM and return to terminal

+ Ignore a file previously tracked: `git rm --cached <file>`
    - This removes the file from the tracking index.  If you omit the `--cached` attribute, you will actually delete the file from your local repo as well.











    Commands to become familiar with from Pellucid:
    Command | Effect
    --------|---------
    `which git` | Shows directory of Git you're linked to
    `brew link [--overwrite] git` | links system to the version of Git installed via brew. Overwriting attribute is optional, was needed for today.
    `. /Users/jpw/.bash_profile` | does something with opening directory of this bash profile? unsure.  Look into
    `git stash [pop]` | adds files to git stash or pops them off.  Not sure of mechanics of this, really.
    `git rebase` | redirects base at which a branch is pointing, I believe.  Base determines what staged files/commits are used in a pull request (or something along these lines).
    `git checkout upstream/master -- test/resources/` | changes branch to upstream/master, but unsure what the test/resources addition does (does it say this is what the upstream/master should be?)
    `git diff upstream/master -- test/resources` | shows diffs between these two branches/stages ?
    `git reset --hard trb_rebase` | A hard reset, but I have no clue what that's really doing.
    `brew install git bash-completion` | install the tab completion ability for terminal use of Git (nice, requires using `brew link git` to be pointing to the modern version of git!)
    `brew install tig` | I do not know what this is at the moment.
    `which tig` | show dir that tig is in
    `git log origin/test_roll_daily` | display log of changes from my testing branch, I believe
    `git merge --abort` | assume back out of the merge ?
    `git merge origin/test_roll_daily` | assume merge my testing branch with origin ?
    `git fetch origin` | ...
    `git diff HEAD -- test` | diffs b/w HEAD and ...?
