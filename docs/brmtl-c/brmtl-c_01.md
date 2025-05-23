# 第一部分

嵌入式编程

让我描述一个“简单”的嵌入式系统。它是一个电池供电的处理器，放置在一个佩戴在某人脖子上的挂坠中。当最终用户遇到紧急情况时，他们按下按钮，计算机会向接收器发送一个无线电信号，从而发出紧急呼叫。

听起来很简单……除了你必须向无线电发送一组精确的脉冲，以便它能生成正确的信号。系统还必须定期检查电池并将电池信息发送给基站，这有两个目的。首先，当电池电量开始降低时，报警公司会收到通知，并向最终用户发送新的挂坠。其次，如果基站没有接收到定期信号，报警公司会知道挂坠出了问题。

这种类型的程序在嵌入式世界中是典型的。它小巧、必须精确，并且不依赖太多外部资源。

在本书的这一部分，你将学习基本的 C 语言语法和编程技巧。我们还会详细讲解 C 语言编译器的工作原理，以便你能精确控制程序的执行。要实现这种精确控制，你需要了解编译器在你不注意时做了什么。

嵌入式编程带来了独特的调试挑战。幸运的是，像 JTAG 调试接口这样的工具使得事情变得更容易，但即便如此，调试嵌入式系统仍然可能相当困难。

最基本且常见的调试方法之一是将`printf`语句放入代码中。这在嵌入式编程中有些困难，因为没有地方可以发送打印输出。我们将介绍如何使用串行输入/输出将打印数据从嵌入式系统中取出，用于调试和日志记录。

最后，在本书的这一部分，你将学习中断编程。中断使得你可以高效地进行输入/输出操作，但如果操作不当，它也可能导致竞态条件和其他随机的错误。在这里，设计至关重要，因为中断问题可能相当难以调试。

欢迎来到嵌入式编程的世界。祝你玩得开心。
