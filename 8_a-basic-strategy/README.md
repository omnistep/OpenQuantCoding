# 10.一个基础的策略

## 从基础代码生成新的OpenQuant策略代码（Solutions）

通常C\#语言开发过程中，我们会从一个我们已经开发的基础应用解决方案\(Solutions\)，形成一个新的程序（解决方案）。这时，我们可以先把原有的Solutions代码目录，完整的复制一份并把目录名定义为未来新的名字，然后用Visual Studio打开，在解决方案资源管理器中，进行重命名操作。

![](../.gitbook/assets/renameyoursolutions.png)

在Visual Studio中，可以方便地对，Solutions，Project，类库等等代码文件方便地进行重命名。

![](../.gitbook/assets/icon_labtubeblue%20%286%29.ico)现在我们以FlashOrder为基础，形成一个新的策略LinesCrossover，开始尝试开发一个真正的交易策略。注意修改主策略代码中的日志标识，以便在未来交易日志中区分新的策略命名和原有的策略命名。

```text
        //定义策略名字 StrategyName，将被记录在日志中
        String StrategyName="FlashOrder要改成LinesCrossover";
```

![](../.gitbook/assets/icon_paw%20%287%29.png)借助Vistual Studio工具，可以快速地基于一个基础的C\#解决方案代码，生成一个新的解决方案。

