PAUP*是一款用于系统发育的商业软件，不过目前可以免费下载到，[下载链接][1]。

<!--more-->

## 2020年+，为什么要使用PAUP*？
  PAUP*与大家熟知的MEGA相比，用户友好程度不高，但是却可是实现少有的最大似然法（ML）分区建树，这一点很少软件能够做到，也是我们容许他速度慢一点的一个“借口”吧。RAxML可以实现在MacOS以及Linux的分区建树，但是考虑到性价比以及软件生态等，假设目前的工作环境为Windows系统。

  同时，PAUP*与TNT、WinClada相比，维护状况较好，支持对于形态学等离散状态的最大简约法建树，虽然DAMBE支持这类数据，但是无法直接输出tre文件，不方便后续树的美化。
## 前期数据的准备与处理
 - 先对数据进行align。可以使用的软件有MAFFT，ClustalW。
 - 对序列保守区的选择，可以手动，也可以不删除。可以使用的软件有GBlocks。
 - 使用DAMBE的Xia et al.的方法对序列进行饱和度检验。
 - 将fas等格式文件转换至nexus文件。可以手动，但是建议使用EasyCodelML这个软件转换，其它如DAMBE或者MEGA等写得过于繁琐。
 - 格式如下最好，名称必须是连续的，一般推荐用_来连接，"-"":""."等不要出现在名称中，并且不要出现中文的各种符号，不然会报错。

## 数据实例

    Begin data;
    Dimensions ntax=4 nchar=361; #ntax=阶元数量 nchar=为位点数量
    Format datatype=DNA gap=- missing=?; #对于PAUP*来说，如果是非DNA等则需要改写成Format datatype=standard gap=- missing=? symbols="123";其中symbols="xyz"取决于你的符号是什么
    Matrix
    Acusta_despecta_no_127                            gctatttctgctcaatgt-ttct-ataaatagccgcagtactttgactgtgcaaaggtagcataatca-attgacttataatt--gaagtctggaatgaaagaatctatggggaaatactgtttcattttgg-tgttaggaaattatt-tattaggtgaaaaaacctataggtaaaaaatagacgagaagacccttgaaattttaattttgttggggcgacaaagtagcaataga-aaacctacttaga--gaatatgtattat--ttataaaggttaaataaattactctagggataacagcataatatttaaaagtttgtgacct-cgatgttgga-ctaggaa-aata-tagtttaga
    Acusta_despecta_no_128                            gctatttctgctcaatgt-ttct-ataaatagccgcagtactttgactgtgcaaaggtagcataatca-attgacttataatt--gaagtctggaatgaaagaatctatggggaaatactgtttcattttgg-tggtaggaaattatt-tattaggtgaaaaaacctataagtaaaaaatagacgagaagacccttgaaattttaattttgttggggcgacaaagtagcaataga-aaacctacttaga--gaatatgtattat--ttataaaggttaaataaattactctagggataacagcataatatttaaaagtttgtgacct-cgatgttgga-ctaggaa-aata-tagtttaga
    Aegista_diversifamilia_CWH_2014_S24_3             ------------------------------------------------------------------------------------------------------------------------------------------taaaattgct-tatcaggtgaaaatacctgactatatataatagacgaaaagaccctggaaatttttattttgttggggcgacagaataacaaat----aacttatttatatataatttgccattt--gtaaataaaataaataaattactccagggataacagcataatatttaaaagtttgtgacct-cgatgttgga-ctaggaa-ttta-tagttcaga
    Amphidromus_contrarius_AM_C_468737                gttttttctgctcaatga-aaat-ttaaatggccgcagtaccctgactgtgcaaaggtagcataatca-gttggcttataatt--gaagtctggaatgaatgaataaacggagggtagctgtgtcttactga-aaccatgaacttattaaagtaagtgaaaatacttacattaaaataatagacgagaagaccctagaaatttgaattttgttggggcgacaaaatagcaagt---taacctatttacg-tgtacaagtgctaaa---ttgtgggtatgaataattactctagggataacagcataatttattaaagattgtgacct-cgatgttgga-ctaggaa-attc-aagttcaga
    Camaena_cicatricosa_GP4                           gcattttctgctcaatga--tat-ttaaatagccgcagtactctgactgtgcaaaggtagcataataa-tttggcttataatt--gaagtcttgtatgaacgaatacatggggaataactatatcaacaatg-taaaatgaaattact-aaatacgtgcaaatacgtatatttacataaaagacgagaagaccctagaaatttttattttgctggggcggcatagtaacatga----aacttacattat-tatacaagaagtgataatttgcagaatgattaaattactctagggataacagcataatttactatagtttgtgacct-cgatgttgga-ttaggaa-gttg-aaatttaga
    Camaena_hahni_isolate_204_1                       gcgatttctgctcaatga--tat-ttaaatagccgcagtactctgactgtgcaaaggtagcataataa-tttggcttataatt--gaagtctagtatgaatgaattcatggggagtaactatatcagcattg-cgagatgaaattact-tattacgtgcaaatacgtatatttaaataaaagacgagaagaccctagaaattttaattttgctggggcggcatagtaacaatg----aacttacaatat-ttgacatgtagcg---gatagcaggacgataaaattactctagggataacagcataatttattatagtttgtgacct-cgatgttgga-ctaggaa-gttg-aaatttaga
    ;
    End;

## 最大简约法（Maximum Parsimony Method）的案例
使用最大简约法，有一个易出错的问题就是往往漏掉bootstrap的数值，直接输出的数值往往比bt出来的数值好看，但是不严谨，所以此处更加推荐bt数值作为支持率。
下面是PAUP*的进行最大简约法分析的bolck，只需要将下述block粘贴到数据block后面即可。#后的注释可以删除。

    begin paup;
    log start replace=yes file=MP.log; #log里面的length一般是需要写到文章里的
    set autoclose=yes;
    set criterion=parsimony;
    set storebrlens=yes;
    set increase=auto;
    hsearch addseq=random nreps=1000 swap=tbr hold=1; #这里的重复数nreps也可以修改，但是没有必要
    savetrees file=MPrandom.tree format=altnex brlens=yes;
    contree all / majrule=yes strict=no treefile=MPcon.tree; #这是综合很多树的合意树
    pscores /tl ci ri rc; #这些数值,CI,RI一般是要写到文章中的
    bootstrap nreps=1000 search=heuristic/ addseq=random nreps=10 swap=tbr hold=1; #bt的nreps(重复数)，一般时以前即可
    savetrees from=1 to=1 file=MPbt.tree format=altnex brlens=yes savebootp=both savebootp=NodeLabels MaxDecimals=0;
    end;

## 最大似然法（Maximum Likelihood Method）的案例
下面是PAUP*的进行最大似然法分析的bolck，只需要将下述block粘贴到数据block后面即可。#后的注释可以删除。

    begin paup;
    log start replace=yes file=ML.log;
    set criterion=like;
    set autoclose=yes;
    set storebrlens=yes;
    set increase=auto;
    charpartition genes=ITS2:1-520, 16S:521-881, 5.8S:882-924; #若不是多基因建树，此行可以删除
    Lset base=(0.1693 0.2836 0.2950) nst=6 rmat=(1.1139 2.7043 1.1663 0.6019 1.9563) rates=gamma shape=0.8030 ncat=4 pinvar=0; #此处的block是由jModelTest计算时，勾选write PAUP*block，得到的结果粘贴而来
    Lset base=(0.3659 0.0887 0.1599) nst=6 rmat=(2.2205 7.0066 3.1324 1.2309 11.1250) rates=gamma shape=0.5450 ncat=4 pinvar=0.2930; #若不是多基因建树，此行可以删除
    Lset base=equal nst=1 rates=equal pinvar=0; #若不是多基因建树，此行可以删除
    hsearch addseq=random nreps=1000 swap=tbr;
    savetrees file=MLrandom.tre format=altnex brlens=yes;
    bootstrap nreps=1000 search=heuristic/ addseq=random nreps=10 swap=tbr hold=1; #第一处的nreps可以修改，但建议不要修改，bt数量小了说明不了问题，大了则计算量过大
    savetrees from=1 to=1 file=MLbt.tre format=altnex brlens=yes savebootp=NodeLabels MaxDecimals=0;
    end;

## 邻接法（Neighbor-Joining Method）的案例
下面是PAUP*的进行邻接法分析的bolck，只需要将下述block粘贴到数据block后面即可。#后的注释可以删除。

    begin paup;
    log start replace=yes file=NJ.log;
    nj;
    set criterion=distance;
    dset distance=ml; #也可以设置为hky85
    bootstrap nreps=1000 brlens=yes treefile=NJdistance.tree search=nj; #第一处的nreps可以修改，但建议不要修改
    savetrees from=1 to=1 file=NJbt.tree savebootp=nodelabels;
    end;

## 运行
在Windows中，打开PAUP*的exe文件，点击菜单栏的File > Open... > 选择的你的文件，注意面板下部应该是exe而不是edit。
## 引用
当您觉得描述太麻烦，也可以引用本文。引用格式示例如下

> Zhang, G. 2020. Usage of PAUP*. malacology.net [Access Time]

  [1]: http://phylosolutions.com/paup-test/
