
### 1. after accept GitHub PR - how to update local - when I'm the owner:
```bash
# (shortest option):
git checkout main
git pull origin main

	# alternative command - exactly the same (longer version)
	git checkout main
	git fetch
	git diff main origin/main
	git merge origin/main


git status
	# make sure no conflicts
```


---
---
---
---



### 2. how to solve merge conflict - when I'm the owner (3 options, they are the same, minor changes):
* option1-shortest:
```bash option1-shortest:
# 1. Make sure your main is up to date
git checkout main
git pull origin main

# 2 - create safe branch there will manually fix merge conflicts
git checkout -b fix_conflict_branch main

git pull https://github.com/DavidSnir/DevOps-ATM-Machine.git main
# Pulls David's main into your current branch (fix_conflict_branch). This is where the conflict will surface.

####### < manually fix conflicts >  #######
####### < manually fix conflicts >  #######
####### < manually fix conflicts >  #######

# 5. commit, and make sure no more conflicts
git add . && git commit -m "fix merge conflict"
git log --oneline -5               # confirm the merge commit is there
git diff main fix_conflict_branch  # should be empty if all is good

# 6. merge to main & push
git switch main
git merge fix_conflict_branch --no-ff -m "Merge fix: resolved conflict with contributor"
git push origin main
```
* option 2 - use FETCH_HEAD raw (exactly like option 3, different naming):
* FETCH_HEAD - where git stores the result temporarily — no remote saved
```bash option 2 - use FETCH_HEAD raw (exactly like option 3, different naming):
# 1. Make sure your main is up to date
git checkout main
git pull origin main

# 2. Fetch contributor's branch WITHOUT merging
git fetch https://github.com/DavidSnir/DevOps-ATM-Machine.git main
	# main - Only the main branch. if not specify will fetch All branches from the remote

# 3. See the diff BEFORE touching anything
git diff HEAD FETCH_HEAD   # what they have that you don't  ← usually what you want
git diff FETCH_HEAD HEAD   # what you have that they don't
subl /tmp/dar_diff_david.txt

# 4. Only NOW create a branch and pull to resolve
git checkout -b fix_conflict_branch
git merge FETCH_HEAD
####### < manually fix conflicts >  #######
####### < manually fix conflicts >  #######
####### < manually fix conflicts >  #######

# 5. commit, and make sure no more conflicts
git add . && git commit -m "fix merge conflict"
git log --oneline -5   # confirm history looks right
git diff main fix_conflict_branch   # should show nothing alarming

# 6. add, commit, merge to main & push
git status
git switch main
git merge --no-ff fix_conflict_branch
git push origin main

# clean up:
git branch -d fix_conflict_branch
```
* option 3 - save at branch "FETCH_HEAD":
```bash option 3 - save at branch "FETCH_HEAD":
# 1. Make sure your main is up to date
git checkout main
git pull origin main

# 2. Fetch contributor's branch WITHOUT merging
git fetch https://github.com/DavidSnir/DevOps-ATM-Machine +main:david_fetched_main
	# main - Only the main branch. if not specify will fetch All branches from the remote
	# now will save it with name, not "FETCH_HEAD" 
	# "+" - if "david_fetched_main" already exist, force updates

# 3. See the diff BEFORE touching anything
git diff main david_fetched_main                         # what David changed vs your main
git diff main david_fetched_main > /tmp/diff_david.txt   # save to file
subl /tmp/diff_david.txt                         	     # open in Sublime

# 4. Only NOW create a branch and pull to resolve
git checkout -b fix_conflict_branch
git merge david_fetched_main

####### < manually fix conflicts >  #######
####### < manually fix conflicts >  #######
####### < manually fix conflicts >  #######

# 6. commit, and make sure no more conflicts
git add . && git commit -m "fix merge conflict"
git log --oneline -5               # confirm the merge commit is there
git diff main fix_conflict_branch  # should be empty if all is good

# 7. merge to main & push
git switch main
git merge fix_conflict_branch
git push origin main

# 8. clean up
git branch -d fix_conflict_branch
git branch -d david_fetched_main
```



---
---
---
---





# extra info:
### contributors never push directly to repo - only with PR

### another layer of protection:
GitHub (Settings → Branches → Branch protection rules) so that even if someone somehow had access to "main", they can't push without a PR.


### If the contributor has an active feature branch and upstream has changed:
contributor should also run "git rebase upstream/main" on that branch (or "merge upstream/main" into it) to avoid a messy PR (that what we did on stage "4.5. problem solve - what if Owner updated before your PR")




----------
----------
----------
## "git push --set-upstream" Vs. "git remote add upstream"
"git remote add upstream" - to track remote
"git push --set-upstream" - track specific branch at remote to specific local branch


"remote add upstream" - "upstream" here is just a name — a convention, not a keyword, You could call it anything. is about where to fetch the owner's code from.

"push --set-upstream" - Git keyword, is about where your branch pushes/pulls (=track) by default.
After setting it, plain "$ git push" / "$ git pull" knows where to go without arguments.
```bash
git push        # instead of: git push origin my-feature
git pull        # instead of: git pull origin my-feature
git status      # shows "ahead 2, behind 1" vs the remote
```
At clone — main is automatically set to track origin/main.
At push with "-u" or "--set-upstream" - both the same.
```bash
git branch --set-upstream-to=origin/my-feature my-feature
	# alternative command (exactly the same - shorter) 
	git push -u origin my-feature
```
