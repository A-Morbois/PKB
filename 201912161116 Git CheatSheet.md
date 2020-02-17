# 201912161116 Git CheatSheet
tags = #Git , #Documentation, #IT


## Branch
* Connaitre les branches présentes dans le répot
`git branch -a `

* Bouger de branche
`git checkout <Branch_Name>`

* Créer une branche 
`git checkout -b <Branch_Name>`




## Commit 
* See last commit
`$ git log --all --oneline --graph --max-count 20`


* modify last commit 
`git commit --amend`


* remove last commit while keeping change
`git reset --soft HEAD~1`

* remove last commit and remove changes
`git reset --hard HEAD~1`


## Work Folder
* See current work folder
`git status`

* Reset your work folder 
`git stash + git stash drop`




