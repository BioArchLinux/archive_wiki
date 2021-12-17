在将序列放入系统发育的计算软件（e.g.MrBayes, RAxML等）前，应当先对序列进行一定的处理，具体流程是先进行比对(MAFFT)，再进行保守区的选择（GBlocks），之后饱和度检验(DAMBE，Tracer)，再是最适模型的选择（jModelTest）。
<!--more-->

## MAFFT的使用
[下载地址][1]
将序列放入MAFFT.bat相同的文件夹中，双击MAFFT.bat

    Active code page: 65001
        
    Preparing environment to run MAFFT on Windows.
    This may take a while, if real-time scanning by anti-virus software is on.                                                                                                                           
    It may take a while before the calculation starts                                                                       
    if being scanned by anti-virus software.                                                                                
    Also consider using a faster version for Windows 10:                                                                    
    https://mafft.cbrc.jp/alignment/software/wsl.html                                                                                                                                                                                               
    ---------------------------------------------------------------------                                                                                                                                                                              
    MAFFT v7.450 (2019/Aug/23)                                                                                                                                                                                                                           
    MBE 30:772-780 (2013), NAR 30:3059-3066 (2002)                                                                          
    https://mafft.cbrc.jp/alignment/software/                                                                       
    ---------------------------------------------------------------------                                                                                                                                                                                                                                                                                                   
    Input file? (FASTA format; Folder="C:\mafft-7.450-win64-signed\mafft-win")                                              
    @    
@后输入文件名称，如`example.fas`

    Output file?
    @ 

@后输入输出名称，一般喜欢加个out，此处键入`exampleout.fas`
  

    Output format?
      1. Clustal format / Sorted
      2. Clustal format / Input order
      3. Fasta format   / Sorted
      4. Fasta format   / Input order
      5. Phylip format  / Sorted
      6. Phylip format  / Input order
    @          
选择`4`

    Strategy?
      1. --auto
      2. FFT-NS-1 (fast)
      3. FFT-NS-2 (default)
      4. G-INS-i  (accurate)
      5. L-INS-i  (accurate)
      6. E-INS-i  (accurate)
    @         
选择`1`      

    Additional arguments? (--ep ## --op ## --kappa ## etc)
    @      
直接`Enter`     

    command=
    "/usr/bin/mafft"  --auto --inputorder "16S.fas" > "16Sout.fas"
    OK?
    @ [Y]  

继续`Enter`，等待结果，到根目录找到example.fas即可



  [1]: https://mafft.cbrc.jp/alignment/software/windows.html

