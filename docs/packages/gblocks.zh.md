在将序列放入系统发育的计算软件（e.g.MrBayes, RAxML等）前，应当先对序列进行一定的处理，具体流程是先进行比对(MAFFT)，再进行保守区的选择（GBlocks），之后饱和度检验(DAMBE，Tracer)，再是最适模型的选择（jModelTest）。
<!--more-->

## GBlocks的使用
[下载地址][2]
将exampleout.fas移入Gblocks.exe的根文件夹，注意序列文件的名称不能过长或者含有一些非法符号，不然会闪退。

    ******************************************************
                        GBLOCKS 0.91b
    SELECTION OF CONSERVED BLOCKS FROM MULTIPLE ALIGNMENTS
            FOR THEIR USE IN PHYLOGENETIC ANALYSIS
    ******************************************************
    
    o. Open File
    
    b. Block Parameters
    
    s. Saving Options
    
    g. (Get Blocks)
    
    q. Quit
    
    
    Your Choice:

选择o

    Please enter a DNA or protein alignment in NBRF/PIR or FASTA format or a paths file.
    File Name:

输入`exampleout.fas`

    ******************************************************
                        GBLOCKS 0.91b
    SELECTION OF CONSERVED BLOCKS FROM MULTIPLE ALIGNMENTS
            FOR THEIR USE IN PHYLOGENETIC ANALYSIS
    ******************************************************
    
    CURRENT FILE: exampleout.fas
    t. Type Of Sequence: Protein
    
    o. Open File
    
    b. Block Parameters
    
    s. Saving Options
    
    g. Get Blocks
    
    q. Quit
    
    
    Your Choice:

此时可以选择`b`更改一些数值

BLOCK PARAMETERS

    1. Minimum Number Of Sequences For A Conserved Position: . 21
    2. Minimum Number Of Sequences For A Flank Position: ..... 34
    3. Maximum Number Of Contiguous Nonconserved Positions: .. 8
    4. Minimum Length Of A Block: ............................ 10
    5. Allowed Gap Positions: ................................ None
    
    r. Restore Defaults
    g. Get Blocks
    
    z. Extended Block Options
    m. Go To Main Menu
    
    
    Your Choice:

输入对应选项可以修改，也可以不修改，此处不修改，直接选择`g`

    exampleout.fas
    Original alignment: 1085 positions
    Gblocks alignment:  81 positions (7 %) in 4 selected block(s)
    
此时根目录里`exampleout.fas-gb.htm`是保守区选择信息，将`exampleout.fas-gb`修改为`exampleoutgb.fas`或者其它以`fas`结尾的文件，得到了选择了保守区的序列。

  [2]: http://molevol.cmima.csic.es/castresana/Gblocks.html

