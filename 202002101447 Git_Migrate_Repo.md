# 202002101447 Git_Migrate_Repo
tags = #Git #IT


* Create a new repo and Initialize it

* Get its new URL

* On the repository to migrate, open the console and clone it to new new one, push it, then set the url to the new

`git clone --mirror <url_new_repo>`

`git push --mirror <url_new_repo>`

`git remote set-url origin  <url_new_repo>`

* Lock the old branch to avoid pushing on the old repo.
