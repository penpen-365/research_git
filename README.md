# research_git
リバートではなくチェックアウトで戻れるか調査

## 課題

b-c-dとコミットが進んだreleaseブランチをa地点に戻したい。
revertだと将来的に`c(誤:developブランチをpull) `の変更がmainに入らないのではないか？
と思い、revert以外でPRを立ててmainブランチをa地点の状態に戻す方法を探した。

```
コミット図：

a --- b(hotfix) --- c(誤:developブランチをpull) --- d(hotfixの追加修正) ← release
 \              
  \--- b'        ← rollback_branch（aと同じ状態の空コミット）
```


## 調査内容

＜調査1＞
コミットハッシュ地点で巻き戻し用ブランチを切って、空コミットでPRを立ててmainへマージ。
```
$ git checkout -b {ブランチ名} {戻りたいコミットハッシュ値}
$ git commit --allow-empty -m "巻き戻しPR"
```
結果は、失敗。巻き戻らず差分なしのコミットとして記録された。


＜調査2＞mainブランチのHEADからブランチを切って、git reset --hardしてから、空コミットでPRを立ててmainへマージ。
```
$ git reset --hard {戻りたいコミットハッシュ値}
$ git commit --allow-empty -m "巻き戻しPR"
```
結果は、失敗。巻き戻らず差分なしのコミットとして記録された。


## 結論

それぞれ失敗した。
PRを通じてソースの状態をコミット前に戻す（打ち消す）方法は`git revert {打ち消したいコミットハッシュ値}`しかなさそう。

複雑なケースになる場合は、
現在のmainブランチを_old_mainなどに名前を変えて`git checkout -b main {戻りたいコミットハッシュ値}`して新しいブランチを作り
それを今後mainとして運用していく、と決めて進めるのがいいかもしれない。※要調査
