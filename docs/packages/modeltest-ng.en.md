# modeltest-ng

基础用法

```
modeltest-ng -d nt -i 输入fas -o 输出名称 -T mrbayes/raxml --force -p 2
```

`-d` 表示数据类型，`nt`表示核酸，`aa`表示蛋白质

`-i` 表示输入文件，可以是`fas`或者`phylip`格式

`-o` 表示输出名称

`-T` 表示模版，有`raxml`, `phyml`, `mrbayes`, `paup` 可以选择

`-p` 表示处理器使用个数

可以在 log 文件中查看输出的最佳模型。
