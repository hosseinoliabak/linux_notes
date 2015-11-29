# My Git notes
* To store credentials: `git config credential.helper store`
* `git config --global --list`
* To show the object type: `git cat-file 56eda069ce7b1b8f113c6107bea69fad264b7765 -t`
* To see the object contents: `git cat-file 56eda069ce7b1b8f113c6107bea69fad264b7765 -p`
* **blob** is only the content of the file
* Stop tracking: `git rm -rf .git`

* Working directory, staging area, and Git directory

![image](https://user-images.githubusercontent.com/31813625/33519789-d976c7b0-d77b-11e7-9799-c7521a5fa353.png)

* Remove anything from staging area: `git reset`
* See information about the remore repository
  * `gti remote -v`
  * `git branch -a`
* Create a branch
  <pre>
  git branch test
  git checkout test
  </pre>
* Merge a branch
  <pre>
  git checkout master
  git pull origin master
  git branch --merged
  </pre>
  Then before going to the next command suits be sure you are in the master branch:
  <pre>
  git merge test
  git push origin master
  </pre>    
* We changed a file and want to cancel the changes since the last commit: `git checkout filename`
* We messed up with commit message. How o modify the commit message? `git commit --amend -m "Message"`
* We left a file without commit and we want to put it as a part of last commit. `git commit --amend`
and then quite the message after saving the message
* We did a commit in a wrong branch. How do we move the commit to the other branch?
For example we committed to the *master* branch but we wanted to commit into the *test* branch
<pre>
git checkout test
git cherry-pick 5f96fa 
git log
git checkout master
</pre>
Then we have to reset.

**Soft reset:** Then we no longer have the last commit. But the files get kept in the staging area
The hash is the hash `7e23bb` of the last commit which should be part of the master branch
<pre>
git reset --soft 7e23bb  
</pre>
**Mixed reset:** IT is the default reset. We have no longer have the last commit and the changes went to the working directory
<pre>
git reset 7e23bb
</pre>
**Hard reset:** You will get rid of the last commit and changes
<pre>
git reset --hard 7e23bb
</pre>
Then we run the `git clean -df` command

**git reflog**
a short cut that's equivalent to: `git reflog show HEAD

**git revert**
It does NOT modify our history. It will created another commit on top of the other commits
but reverts the changes.
<pre>
git revert 58cc49f
</pre>

**git stash**
<pre>
git stash save "Worked on test, will apply later to the file"
git stash list
</pre>
* I practiced `git stash apply stash@{0}`, `git stash pop`, `git stash drop stash@{0}`, `git stash clear`
* git stash is also good to carry over from branch to branch. We checkout to the *test* branch
then we pop the stashed to the *test* branch
