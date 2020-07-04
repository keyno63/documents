# Fork

Git branch の fork をしたときの管理.  
fork したものの、リモートブランチが更新されたときに追従が必要になる.  
そのときの fork 先 repository 更新をどうするかのメモ.  

## fork
Github で branch を fork する.

## local 設定  
local repository の準備.
```
git clone git@github.com:<myaccount>/<repository>.git
```

## root branch 設定

remote 設定に root branch(fork 元)の設定をする.
```
git remote add root_branch git@github.com:<org>/<repository>.git
```

## fork branch の更新

fork 元で行われた更新を fork 先の repository に反映させる.
```
// root branch の変更を local repository にとりこむ
git fetch root_branch

// local repository の working branch に更新
git merge root_branch/master

// remote repository に繁栄
git push origin master
```