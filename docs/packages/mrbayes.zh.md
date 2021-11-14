MrBayes是贝叶斯推断法(Bayesian Inference)构建系统发育的经典软件，[下载地址点击此处。][1]
<!--more-->

## 前期数据的准备与处理
 - 先对数据进行align。可以使用的软件有MAFFT，ClustalW。
 - 对序列保守区的选择，可以手动，也可以不删除。可以使用的软件有GBlocks。 使用DAMBE的Xia et al.的方法对序列进行饱和度检验。
 - 将fas等格式文件转换至nexus文件。可以手动，但是建议使用EasyCodelML这个软件转换，其它如DAMBE或者MEGA等写得过于繁琐。
 - 格式如下最好，名称必须是连续的，一般推荐用_来连接，"-"":""."等不要出现在名称中，并且不要出现中文的各种符号，不然会报错。数据实例参见[此处][2]。
 - 使用jModelTest基于BIC数值对模型进行选择。BIC数值越低越适合。

## 单基因建树
下面是MrBayes的进行BI分析的bolck，只需要将下述block粘贴到数据block后面即可。#后的注释可以删除。

    begin mrbayes;
    lset nst=2 rates=gamma; #模型，nst=1/2/6 (F81/HKY/GTR); +I=propinv,+G=gamma,+G+I=invgamma
    Prset statefreqpr=dirichlet(1,1,1,1);
    mcmcp savebrlens=yes ngen=2000000 samplefreq=100 nchains=4; #ngen(number of generatoins)可以调整，不过不足就继续添加，如果足够可以提前结束
    mcmc;
    sump;
    sumt contype=allcompat burnin=5000;
    end;

## 多基因建树
下面是MrBayes的进行BI分析的bolck，只需要将下述block粘贴到数据block后面即可。#后的注释可以删除。

    begin mrbayes;
    outgroup 5406.1; #可以设置也可以不设置，若不设置，这一行删除
    CHARSET ITS = 1-1027; #后面名称与之一致即可,区段根据实际修改
    CHARSET 16S = 1028-1391;
    partition gene=2:ITS,16S;
    set partition=gene;
    lset applyto=(1) nst=2 rates=propinv; #对于第一个基因，nst=1/2/6 (F81/HKY/GTR)
    lset applyto=(2) nst=6 rates=gamma; #+I=propinv,+G=gamma,+G+I=invgamma
    Prset applyto=(all)statefreqpr=dirichlet(1,1,1,1);
    mcmcp savebrlens=yes ngen=2000000 samplefreq=100 nchains=4; #ngen(number of generatoins)可以调整，不过不足就继续添加，如果足够可以提前结束
    mcmc;
    sumt contype=allcompat burnin=5000;
    end;
    
## 形态学数据建树
目前MrBayes仅仅支持简单的MK模型类似与基因中的JC模型，形态学建树更多采取MP方法，详见[PAUP*][3]。不过，BI法在形态中也是尤其优越性，可以作为一个比对结果。

与PAUP*相比，MrBayes的datatype为standrd所能定义的状态要少。下面是一个形态学的实例。

    Begin data;
    	Dimensions ntax=25 nchar=28;
    	Format datatype=Standard gap=- missing=?;
    	Matrix
    Camaena              0000011200120222223322222220
    Mastigeulota         0001000001012101100000000000
    Metodontia           0001000000012101000200000000
    Fruticicola          1211100000000111000210001000
    Acusta               0001211200000111111210000100
    Bradybaena           1121000000011101000200000000
    Karaftohelix         1211111200000011000001000000
    Cathaica             0001000000011110100000000000
    Pliocathaica         0001011200000111100001000000
    Aegista              0001011200113111000200000000
    Plectotropis         0001011200113110000200000000
    Pseudaspasita        0001011200111111000200000000
    Platypetasus         0001000000000101100000000000
    Stilpnodiscus_spa    0001100000000111100000000000
    Stilpnodiscus_spb    0001100000000101100000000000
    Laeocathaica         0001211200000111100000000000
    Pseudobuliminus      0001000000000101100200000000
    Trishoplita          0001011200100011112000000000
    Euhadra              0001011200100011112000000000
    Nesiohelix           0000011200100011111000010000
    Aegistohadra         0000011210100111100200000010
    Eueuhadra            0000011210100111100000100000
    Calocochlea          1120012100000111100110000000
    Pfeifferia           1120012100000111100110000001
    Trichobradybaena     1121000001000101100200000000
    	;
    End;
其后面的MrBayes的Block如下

    begin mrbayes;
    mcmcp savebrlens=yes ngen=2000000 samplefreq=100 nchains=4; #ngen(number of generatoins)可以调整，不过不足就继续添加，如果足够可以提前结束
    mcmc;
    sump;
    sumt contype=allcompat burnin=5000;
    end;

## 形态学和分子数据建树
本人不建议使用二者的结合，因为随着序列的增长，形态学数据占据的比例会急剧下降，这对于形态学数据来说是一个不公的处理。
下面是数据示例。

    Begin data;
    Dimensions ntax=25 nchar=100;
    Format datatype=mixed(Standard:1-28,DNA:29-100) interleave=yes gap=- missing=?; #根据自己的数据调节
    Matrix
    Camaena              0000011200120222223322222220CATTCGACCCGATGAGATTAGCAGTATCATTCGTAAACAAATTGAAGGATATGTTCCAGAAGTAAAGGTTGT
    Mastigeulota         0001000001012101100000000000CATTCGACCCGATGAGATTAGCAGTATCATTCGTAAACAAATTGAAGGATATGTTCCAGAAGTAAAGGTTGT
    Metodontia           0001000000012101000200000000CATTCGACCCGATGAGATTAGCAGTATCATTCGTAAACAAATTGAAGGATATGTTCCAGAAGTAAAGGTTGT
    Fruticicola          1211100000000111000210001000CATTCGACCCGATGAGATTAGCAGTATCATTCGTAAACAAATTGAAGGATATGTTCCAGAAGTAAAGGTTGT
    Acusta               0001211200000111111210000100CATTCGACCCGATGAGATTAGCAGTATCATTCGTAAACAAATTGAAGGATATGTTCCAGAAGTAAAGGTTGT
    end;

下面是MrBayes的Block。

    begin mrbayes;
    CHARSET morphology = 1-28; #根据自己的数据调节
    CHARSET gene = 29-100;
    partition favored=2:morphology,gene; #根据自己的命名调节
    set partition=favored;
    lset applyto=(1) nst=6 rates=invgamma; #根据自己的分子模型调整
    lset applyto=(2) rates=gamma; #+G=gamma,如果单纯的MK模型,那rates=equal
    prset applyto=(all) ratepr=variable;
    mcmcp savebrlens=yes ngen=2000000 samplefreq=100 nchains=4; #ngen自行调整
    mcmc;
    sumt contype=allcompat burnin=5000;
    end;

## 运行
点击exe文件，输入`exe 你的文件名`。如果你的文件和exe处于同一个文件夹，那么直接输入即可，如果不是，就需要输入路径。
当Average standard deviation of split frequencies持续小于0.01时，那么，使用Ctrl+c结束，会有话语Do you really want to stop the run (y/n)?出现，输入y即可。
## 树的选择
选择`[你的输入文件名].con.tre`打开，进行树的美化。
## 引用
当您觉得描述太麻烦，也可以引用本文。引用格式示例如下

> Zhang, G. 2020. Usage of MrBayes. malacology.net [Access Time]

  [1]: https://nbisweden.github.io/MrBayes/download.html
  [2]: https://www.malacology.net/index.php/archives/18/#%E6%95%B0%E6%8D%AE%E5%AE%9E%E4%BE%8B
  [3]: https://www.malacology.net/index.php/archives/18/
