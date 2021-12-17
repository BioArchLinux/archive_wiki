# raxml-ng

基本使用

```
raxml-ng --all --msa fas文件 --model GTR+G4 --prefix 名称 --bs-trees 1000
```

`-msa` 比对好的序列文件

`--model` 模型，可以参照 modeltest-ng 的输出

`--prefix` 前缀

`--bs-trees` BootStrap 树的个数
