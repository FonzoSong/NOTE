####  关于git pull

![gitPullError1](https://raw.githubusercontent.com/FonzoSong/NoteImages/master/img/gitPullError1.svg)在多人编辑同一分支时，用户A推送了更改，用户B推送时会遇到：

``` bash
! [rejected] master -> master (non-fast-forward)
```

原因就是 revise A 与 B 冲突，可以直接pull 一下完事。

![gitPullError2](https://raw.githubusercontent.com/FonzoSong/NoteImages/master/img/gitPullError2.svg)

但会导致历史数据混乱（会有很多的无用merge记录），在pull时添加 `--rebase` 来保持线性树整洁；

![gitPullError3](https://raw.githubusercontent.com/FonzoSong/NoteImages/master/img/gitPullError3.svg)

默认配置：`git pull.rebase true`
