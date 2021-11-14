# jModelTest的使用

[下载地址][4]，必须在java环境下运行。

菜单栏`File` > `Load DNA Alignment`选择你的文件。

菜单栏`Analysis` > `Compute Likelihood Scores`，因为用不到太多模型，也为了提高速度，将`Number of Substitution Schemes`改为`3`再点击`Compute Likelihoods`。

菜单栏`Analysis` > `Do AIC calculations`全选后点击`Do AICc calculations`选择ML的最适模型。

菜单栏`Analysis` > `Do BIC calculations`全选后点击`Do BIC calculations`选择BI的最适模型。

菜单栏`Results` > `Show Results Table`根据AICc和BIC查看最适模型。

菜单栏`Results` > `Build HTML log`保存文件。


  [4]: https://github.com/ddarriba/jmodeltest2
