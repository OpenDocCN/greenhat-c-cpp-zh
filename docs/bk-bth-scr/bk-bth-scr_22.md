

## 第二十章：20 个高级`for`技巧



![](img/chapter.jpg)

过去的三章展示了`for`命令的强大功能，但也演示了它使用起来的复杂性，甚至揭示了一些限制。在本章中，我将探讨一些高级主题，包括如何绕过这些限制的技巧。

在本章中，我将解释如何执行以下任务：

+   让解释器在`for /F`命令查询的数据中，尊重连续分隔符字符之间的空值。

+   使用传统的 Batch，或者通过将 PowerShell 或 Python 命令嵌入 Batch 代码中，强制将字符串转换为大写（或小写），这是一种可以扩展到其他命令、语言和应用程序的技巧。

+   在`for`命令的代码块中实现两个级别的延迟扩展

+   在`for`命令中处理转义，尤其是使用`delims`子句中的双引号。

这些技巧将使你能够处理更多类型的数据和更复杂的变量。如果你编写批处理文件的时间足够长，至少其中的几个话题对你来说会变得相关。更重要的是，它们应该展示了一种解决问题的方法，这将激发你在遇到其他未知问题时的想象力。

### 尊重空值

在第十九章中，我提到过，当使用`delims`子句将字符串分割为独立标记时，解释器不会尊重空值。考虑一个名为`staff.csv`的文件，其中包含每个员工记录的五个项目：名字、中间名、姓氏，后跟职位和政府 ID：

```
Amy,Amanda,Andersen,Architect,111-11-1111
Colin,,Clark,Coder,222-22-2222
Mai,Maria,McManus,Manager,333-33-3333 
```

数据组织得很好，但可读性不强。然而，以下代码会将来自前四个标记的数据以基础报告的形式显示到控制台，同时不显示记录末尾的敏感数据——或者至少是这样的意图：

```
set cnt=0
for /F "usebackq tokens=1-4 delims=," %%a in ("C:\Batch\staff.csv") do (
   set /A cnt += 1
   > con echo Employee !cnt!:
   > con echo    First Name: %%a
   > con echo   Middle Name: %%b
   > con echo     Last Name: %%c
   > con echo     Job Title: %%d
) 
```

前两条记录生成如下内容，但请注意，这里存在一个重大问题：

```
Employee 1:
   First Name: Amy
  Middle Name: Amanda
    Last Name: Andersen
    Job Title: Architect
Employee 2:
   First Name: Colin
  Middle Name: Clark
    Last Name: Coder
    Job Title: 222-22-2222 
```

正如你在数据`Colin,,Clark`中看到的，Colin 没有中间名，但`for /F`命令会跳过两个逗号之间的空值，导致后续数据出现偏差，并暴露了 Colin 的政府 ID。在许多情况下，比如在按多个空格分隔时，作为程序员，你可能希望这种行为，但在这里就不行。当遇到这样的数据时，你需要一种方法来强制 Batch 尊重空值。

这个解决方案通过使用嵌套的`for /F`命令，在连续的逗号分隔符之间插入一个空格：

```
set cnt=0
❶ for /F "usebackq tokens=*" %%r in ("C:\Batch\staff.csv") do (
   set inRec=%%r
 ❷ for /F "tokens=1-4 delims=," %%a in ("!inRec:,,=, ,!") do (
      set /A cnt += 1
      > con echo Employee !cnt!:
      > con echo    First Name: %%a
    ❸ > con echo   Middle Name: %%b
      > con echo     Last Name: %%c
      > con echo     Job Title: %%d
)  ) 
```

在外层 for /F 命令❶的开始，我将整个记录存储在 inRec 变量中，然后将它作为内层 for /F 命令❷的文本输入，但请仔细查看这个输入：!inRec:,,=, ,!。首先，我通过感叹号解析 inRec，因为我已经在代码块中为它赋值。更重要的是，这个语法将双逗号替换为由空格分隔的两个逗号，实际上插入了一个空格。内层 for /F 命令❷将四个以逗号分隔的标记传递到它的代码块中，当处理科林的数据时，插入的空格变成了第二个标记%%b，并将其作为中间名❸与其他信息一起写入控制台。

艾米的报告没有变化，科林的条目看起来好多了：

```
Employee 2:
   First Name: Colin
  Middle Name: 
    Last Name: Clark
    Job Title: Coder 
```

他的中间名正确地显示为空，而他的政府身份证明被隐藏起来。

尽管这些嵌套命令对于该特定数据有效，但逻辑远非万无一失。当记录中的第一个标记是空值时，它就无法正常工作，因为这样的空值不会表现为双逗号；空值作为第一个标记时，只表现为一个逗号引领记录，后面跟着第二个标记。它也无法捕捉到两个连续的空值，或是连续三个逗号，在第二个逗号前插入一个空格，但在它后面却没有插入空格。最后，这些嵌套的 for 命令将空值更改为空格，因此它被改变了。根据你如何使用数据，这通常不是问题，但有时你会希望保持数据的完整性。

所有这些规定可能看起来会使这种方法不适用，但如果你了解你的数据，它可能是完全可以接受的。例如，在这个示例数据中，如果你确信中间名是唯一可能缺失的标记，并且你可以接受在中间名标签后写入一个额外的空格，那么这个语法就完全没问题。但总是有改进的空间。

对之前解决方案做一些调整，下面的代码修正了刚才讨论的所有局限性：

```
set cnt=0
❶ for /F "usebackq tokens=*" %%r in ("C:\Batch\staff.csv") do (
   set inRec=%%r
 ❷ for /F "tokens=1-4 delims=," %%a in ("_!inRec:,=,_!") do (
    ❸ set a=%%a& set b=%%b& set c=%%c& set d=%%d
      set /A cnt += 1
      > con echo Employee !cnt!:
    ❹ > con echo     First Name: !a:~1!
      > con echo   Middle Name: !b:~1!
      > con echo     Last Name: !c:~1!
      > con echo     Job Title: !d:~1!
)  ) 
```

外层 for /F 命令❶保持不变。内层 for /F 命令❷仍然传递四个以逗号分隔的标记，但它的输入(_!inRec:,=,_!)在每个标记前加上了一个下划线。这个前置的下划线实际上是对整个记录进行前置操作，只是将第一个标记前加了一个下划线。然后，替换语法将每个逗号改为一个逗号后跟下划线，这会在每个逗号后插入下划线，从而有效地使每个剩余的标记前加上下划线。空值标记现在是一个下划线(,_)，所以解释器会尊重它。

虽然这解决了一个问题，但也引发了另一个问题；我们必须在使用数据项或令牌之前，去除每个数据项或令牌前面新添加的下划线。我不常在单行代码中合并多个命令，但在这里我使用了四个 set 命令，因为它们简短、简单且重复❸。第一个命令，set a=%%a，将令牌 %%a 赋值给变量 a，之后我可以在代码块中通过 !a:~1! ❹ 截取第一个字符，留下只有去除前导下划线后的数据，即原始内容。接着，我对其他三个令牌使用相同的技巧，因为我确信每个令牌前都有下划线。

这种在每个令牌前添加一个字符然后去除它的技巧要更加通用。

### 强制将字符串转换为大写

如果你还没有意识到，我非常喜欢 Batch 编程，但有时其他编程语言能提供与我在 Batch 中构思的不同，甚至更好的解决方案。例如，强制将文本转换为大写在现代语言中是一个微不足道的操作，这些语言有某种可调用的方法，比如 PowerShell 中的 .toUpper() 或 Python 中的 .upper()。不幸的是，Batch 并没有类似的命令或任何内建机制来将文本转换为大写，但我会展示一些不同的方法来完成这个任务。

到现在为止，你已经看到我们如何从头开始构建一些东西，而在这个解决方案中，我们将利用 Batch 中固有的不区分大小写的特性，创建一个实际改变文本大小写的例程。以下例程接受一个字符串变量名作为输入，并返回该变量，其中内容被强制转换为大写：

```
:ToUpper
 for %%u in (A B C D E F G H I J K L M N O P Q R S T U V W X Y Z) do (
    set %~1=!%~1:%%u=%%u!
 )
 goto :eof 
```

这个不带选项的 for 命令执行 set 命令，以延迟扩展的方式更新变量 26 次，每次处理字母表中的一个字符，每次传递都将 %%u 解析为一个大写字符。（也许我应该打破惯例，在这里使用 %%U。）

第一次传递将 A 改为 A，这看起来有些多余，直到你记得批处理文本替换语法是不区分大小写的，这意味着这个技巧同时将 A 和 a 都改为 A。接下来，我们将 B 改为 B，C 改为 C，依此类推，直到整个字母表都被替换完。

当 for 循环完成时，它将所有小写字母转换为大写字母，而不会影响现有的大写字母和非字母字符。传递给这个逻辑的变量作为唯一参数，现在包含的是转换为大写的文本。

你可以创建一个类似的例程，将字符串强制转换为小写，只需将输入的值列表更改为所有小写字符的集合。唯一可能需要更改的是标签，你可能需要将其更新为类似 :ToLower 的名称。

#### 嵌入 PowerShell 命令

另一种解决方案是将前面提到的 PowerShell 命令作为输入传递给 Batch 的 for /F 命令。没错，你没看错；你可以将其他编程语言的逻辑嵌入到 Batch 代码中。但是在将 PowerShell 命令作为 for /F 命令的一部分之前，我们先来看一下它在命令提示符下的运行情况。

如果你的 Windows 电脑中预装了 PowerShell，你可以在命令提示符下执行 PowerShell 命令，除非你在第一代 iPhone 发布之前购买了电脑，否则自 XP SP2（大约 2006 年）以来，每个 Windows 操作系统都会自带 PowerShell。

在命令提示符下输入此命令：

```
**powershell 'Set this to Upper-Case'.toUpper^(^)**
```

解释器将单引号内的文本输出到控制台，所有字母字符都会被转为大写。

一眼看去，这看起来像是一个名为 powershell 的 Batch 命令，但并没有这样的命令。实际上，这是程序 *powershell.exe*，解释器应该能够在路径层级中找到它（大多数电脑上的路径为 *C:\Windows\System32\WindowsPowerShell\v1.0\*）。该程序的参数是 PowerShell 命令，用于将文本转换为大写，这个命令恰好使用了点表示法，正好与传统 Batch 命令相反。被强制转换为大写的文本位于单引号内，后面跟着点和适当的 PowerShell 方法，其括号已转义。

你还可以将此命令放入 bat 文件中，它在你希望看到“SET THIS TO UPPER-CASE”写入标准输出时表现良好，但如果你希望将此文本赋值给一个变量，那么就需要使用 for /F 命令：

```
:ToUpper
 for /F "usebackq tokens=*" %%u in (`powershell '!%~1!'.toUpper^(^)`) do (
    set %~1=%%u
 )
 goto :eof 
```

与第一个解决方案一样，这个例程有一个参数，作为输入和输出使用——也就是包含文本的变量的名称。

包围在反引号中的 for /F 命令的输入告诉解释器这是一个命令输入，在本例中是嵌入的 PowerShell 命令。请注意，这个命令看起来非常像我们之前在命令提示符下输入的命令。唯一的区别是现在 !%~1! 替代了硬编码的文本。%~1 参数解析为输入变量名；然后，感叹号和延迟扩展会将变量解析为其值。PowerShell 命令的输出是该文本的大写版本，解释器将其作为变量 %%u 发送到代码块中，最终将其重新赋值给相同的输入参数：%~1。

#### 嵌入 Python 命令

你不必止步于 PowerShell 命令。任何你可以在命令提示符下输入的内容，都可以作为 for /F 命令的有效输入，包括调用任何程序，而不仅仅是 Batch 命令。在这个示例中，我将调用 Python 运行时，但只要你能够在命令提示符下执行它，你就可以使用任何语言的运行时。你甚至可以执行用其他语言编写并编译的程序，将它的输出作为 for /F 命令的输入。

与 PowerShell 不同，你的 Windows 计算机可能没有预装 Python 运行时，但你可以从 *[`<wbr>www<wbr>.python<wbr>.org<wbr>/downloads<wbr>/`](https://www.python.org/downloads/)* 下载。这里不是一本关于 Python 的书（虽然 No Starch Press 的确有一些很棒的书籍），所以我不会详细探讨语法，但只要你的计算机上安装了 Python，下面的例程在功能上等同于前两个具有相同标签的例程：

```
:ToUpper
 set subopts=usebackq tokens=*
 for /F "%subopts%" %%u in (`python -c ^"print^('!%~1!'.upper^(^)^)^"`) do (
    set %~1=%%u
 )
 goto :eof 
```

我答应不深入探讨，但简而言之，python 是没有扩展名的可执行文件或运行时，-c 告诉运行时接下来是一个 Python 命令。`print()` 函数输出的是括号内内容的结果，该内容是解析后的输入变量，包含单引号、一个点和将字符串转换为大写的 Python 方法。去掉六个转义字符后，Python 命令更容易阅读：`print('!%~1!'.upper())`。

代码块中的 set 命令再次将嵌入命令的输出赋值给它的唯一参数。但还要注意，我设置了一个包含 usebackq 关键字和 tokens 子句的文本变量，然后将该文本作为 for /F 命令的一部分进行解析。这样做有两个原因。第一个也是最直接的原因是，否则命令就放不下在页面上（尽管它是完全有效的）。第二，这是另一个机会，让我提醒你，我们如何在一个未编译的脚本语言中将命令拼接在一起。

实际的 PowerShell 和 Python 命令本身当然很简单，但使用它们所需的 Batch 机制确实增加了一些复杂性。这通常不是解决问题时首先想到的技巧，但当你能够真正利用其他语言的命令时，for /F 命令为你提供了一种有效的方式来分配其输出。

> 注意

*在第十九章中，为了保持一致性，我建议养成在命令作为 for /F 命令输入时使用 usebackq 的习惯。我推荐的部分原因是，虽然在这样的命令中通常找不到反引号或单引号，但单引号是两者中更可能出现的。对于这最后两个例子，关键字和包含反引号不仅是好的实践；它们是必须的，因为嵌入的 PowerShell 和 Python 命令都包含单引号。*

### 延迟扩展的两个层次

延迟扩展是 Batch 的一大特点，其他大多数语言中都没有。当你积累经验并开始构建一个更有趣且复杂的代码库时，你会遇到一个相当常见的问题，即如何在 for 命令的代码块中处理两个层次的延迟扩展。

在第三章中，我通过定义包含五个城市美食的变量来演示了延迟扩展。这里有两个例子：

```
set foodNash=Hot Chicken
set foodNYC=Thin Crust Pizza 
```

使用相同的缩写，我还设置了每个城市完整名称的变量：

```
set NashFull=Nashville
set NYCFull=New York City 
```

最终，以下的 echo 命令包含了两个延迟扩展的示例，这些示例解析了这些变量，前提是 city 变量被分配了有效的城市缩写：

```
> con echo The best !food%city%! can be found only in !%city%Full!.
```

现在，让我们基于这个例子，列出美国所有 50 个州，并为每个州指定一个烹饪之都。例如，虽然杰斐逊市是密苏里州的首府，但圣路易斯以其冷冻卡士达闻名，许多人认为它是该州的烹饪之都。在我成长的州——康涅狄格州，纽黑文独特的薄脆比萨和米德尔顿的蒸奶酪汉堡之间长期以来一直存在争论。我并不想卷入其中，但为了本练习的需要，我（某个权威机构）被赋予了定义每个州烹饪之都的权力，并且明确了每个城市，进而确定该州的代表性菜肴。给定一个州的邮政缩写代码列表，我可以这样定义每个州的烹饪之都：

```
set culCapTN=Nash
set culCapNY=NYC
set culCapIL=Chic
set culCapLA=NO
set culCapMO=STL 
```

例如，TN（田纳西州）的烹饪之都 culCapTN 是 Nash（纳什维尔）。现在，我们可以使用两位数的州代码作为输入，进行 for 命令，其中延迟扩展将烹饪之都变量解析为城市缩写。然后，第二级的延迟扩展将这些缩写解析为食物和完整的城市名称。

这些州的代码可能来自文件中的数据，但为了让这个练习尽可能简单，我使用了硬编码的州代码列表作为没有选项的 for 命令的输入：

```
for %%s in (TN NY IL LA MO) do (
   set city=!culCap%%s!
   > con echo The best !food%city%! can be found only in !%city%Full!.
) 
```

%%s 变量会在五次迭代中解决到州的代码。在数据的第一次迭代中，!culCap%%s! 会解决为 !culCapTN!，然后它会解决为 Nash（纳什维尔），并且我们将其分配给 city 变量。然后，echo 命令从第三章中按字面意思提取，解决了两个延迟扩展的示例，就像在上一章中那样，对吧？

错了！如果这种情况有效，我就不会称其为“高级技巧”了。虽然解释器成功地为 city 变量分配了城市缩写，但当代码处于像这样的代码块内时，变量会有两个不同的值：感叹号定界符将其解决为代码块中已取得的值，而百分号定界符则将其解决为代码块执行之前的值。

因此，%city% 解决为无（或它在先前代码中设置的任何值），然后 !food%city%! 解决为 !food!，而 !food! 很可能也解决为无。代码不会挂起或抛出中止错误——更糟糕的是，它只会显示一个不完整的句子。

我们陷入了困境。由于 city 是在代码块内部设置的，解决它当前值的唯一方法涉及感叹号，但延迟展开要求我们用百分号来解析内部变量。（当首次遇到这个困境时，大多数程序员，如果不是所有程序员，都会尝试交换定界符。你可以试试，但它就是不行。）以某种方式，你必须通过百分号让 city 可解析。

一种解决方案涉及一个隐藏的例程。这个技巧并不会将第一次延迟展开的结果赋值给像 city 这样的变量；相反，它将值传递给一个例程：

```
for %%s in (TN NY IL LA MO) do (
   call :WriteFoodText "!culCap%%s!"
)
if 0 equ 1 (
  :WriteFoodText
   > con echo The best !food%~1! can be found only in !%~1Full!.
   goto :eof
) 
```

例程中的代码随后使用 %~1 语法将城市解析为一个简单的参数。由于例程在 for 命令的代码块外部执行解析，变量 !food%~1! 会很好地解析为 !foodTN!，再次解析为热鸡肉。

然而，这种技巧也不是没有问题的。它不是那种通常适合放在 bat 文件末尾的复杂例程，而且绝对不会单独出现在一个 bat 文件中。它其实只是一个单一的命令（加上标签和终止的 goto :eof 命令），所以最好将其靠近 call 命令。但如果我将标签紧接在这个 for 命令后面，解释器会对每个州执行一次例程，然后在 for 命令完成并控制流转到下一个命令时，再执行一次。为了解决这个问题，我使用了一个永远不会为真的笨拙条件语句 0 equ 1，它作为例程的门卫。

这种技巧很容易让任何阅读代码的人感到困惑，但它成功地隐藏了一个放在 call 命令附近的隐藏例程，而这是访问它的唯一方法。它有效，我经常使用它，但这不是你会想挂在冰箱上和孩子的艺术作品一起展示的那种逻辑。

更为优雅的解决方案是将第二个 for 命令嵌套在第一个命令中：

```
for %%s in (TN NY IL LA MO) do (
   for /F "tokens=*" %%c in ("!culCap%%s!") do (
     > con echo The best !food%%c! can be found only in !%%cFull!.
)  ) 
```

首先，%%s 解析为州代码；然后 !culCap%%s!，解析为相应的城市缩写，是传入内部 for 命令的字符串，我们将其赋值给 %%c。因此，内部的 for 命令只不过是将 city 赋值给一个变量，我们可以通过百分号来解析它。注意，我已经将问题陈述中解析为 null 的 %city% 替换为 %%c 变量。同样，以田纳西州为例，%%c 解析为 Nash，然后 !food%%c! 变为 !foodNash!，进一步解析为美味的热鸡肉。类似地，!%%cFull! 解析为 !NashFull!，并解析为 Nashville。现在，这可是值得贴在冰箱上的代码！

我使用`/F`选项与内部`for`命令配合，并用双引号将文本输入括起来。将 tokens 关键字设置为星号确保`%%c`解析为整个文本字符串，即使它包含空格，这意味着代码块只会执行一次。（一个更简单的不带选项的`for`命令对于这个特定数据也能作为内部`for`命令使用，但`for /F`命令是一个更通用的解决方案，因为它可以处理包含空格和不包含空格的输入。）

本节中详细介绍的两种解决方案会将以下输出写入控制台：

```
The best Hot Chicken can be found only in Nashville.
The best Thin Crust Pizza can be found only in New York City.
The best Deep Dish Pizza can be found only in Chicago.
The best Muffuletta Sandwich can be found only in New Orleans.
The best Frozen Custard can be found only in St Louis. 
```

我提供的这个问题陈述看起来可能有些牵强，但这种技术的需求常常出现。更现实的例子通常涉及更复杂的情况，涉及文件、数组、哈希表等更多我尚未深入探讨的主题。例如，若要为`robocopy`命令获取目标路径，你可能需要从文件中读取目标城市，然后在哈希表中查找该城市以获得路径。但这些情况归根结底都是同一个问题：你需要在代码块中使用延迟扩展来处理之前设置的变量。要时刻留意类似的情况。

### 使用`for`命令进行转义

在`for`命令中转义特殊字符是一个挑战，可能以多种不同方式表现出来，其中一个例子就是在子选项子句中。输入数据通常完全不在我们程序员的掌控之中。如果所有数据文件都是以逗号或管道符分隔的那该多好，但很多时候你不得不使用更复杂的分隔符。最糟糕的情况是文本以双引号分隔。如果你试图提取六个标记，你可能会考虑这个子选项子句，但它是无效的：

```
"tokens=1-6 delims=""
```

虽然解释器正确地将第一个双引号识别为子句的开始，但第二个双引号终止了子句，第三个双引号则处于上下文之外。但这个看起来晦涩的子句完成了任务：

```
tokens^=1-6^ delims^=^"
```

没有双引号开启子句，且结尾的双引号并没有关闭子句——它是分隔符集中的唯一项。包围子句的双引号不再存在。这允许你在`delims`子句中包含双引号，只要你用前导插入符号进行转义。但是，缺少包围选项子句的双引号会引发其他问题。等号甚至空格不再受到包围双引号的保护，现在也需要转义。

我在两个等号前面都加了插入符号（^）。这两个关键字子句之间的空格会终止较大的子选项子句，因此这个空格也需要转义。这个插入符号看起来似乎跟在 6 后面，但实际上它是引导空格的。在批处理的一个不太文档化的特性中，只要你转义了特殊字符和空格，你就可以省略包围的双引号。

现在让我们将这一切结合起来并进行测试。考虑这个非常小的文件，*UglyData.txt*，其中仅有一行数据，用五个双引号分隔六个单词：

```
This"is"some"messed"up"data
```

这个 for /F 命令包含了前面讨论过的选项子句：

```
for /F tokens^=1-6^ delims^=^" %%a in (C:\Batch\UglyData.txt) do (
   > con echo %%a %%f %%b no longer %%d %%e.
) 
```

这是输出到控制台的内容：

```
This data is no longer messed up.
```

在 第十九章 中，我提到过我通常在读取文件时使用 usebackq。这里没有使用，因为子选项已经变得很混乱，但如果你把双引号放在输入文件周围，你需要在 tokens 子句前添加 usebackq^ 和一个空格。（你稍后会看到类似的内容。）

当存在多个分隔符时，相同的原理也适用。让我们通过添加插入符号、管道符、百分号和与符号分隔符，使*UglyData.txt*变得更加混乱：

```
This^is|some%messed"up&data
```

现在，以下的子选项子句使得这个操作得以实现：

```
tokens^=1-6^ delims^=^"^|%%^^^&
```

一个插入符号转义所有这些特殊字符（甚至是插入符号分隔符），除了一个；我们只能通过另一个百分号来转义百分号。

在分隔符集里可能很少会出现双引号的需求，但一旦需要，这项技术就显得不可或缺。

我们还没有完全完成转义。在接下来的示例中，我将使用相同的数据并将相同的输出写入控制台，但数据的来源会不同。我将从一个文件中提取混乱数据，并将其设置为一个变量，这将要求对数据和 for /F 命令进行更改。

我将从数据开始。在 第十四章 中，你学到了设置变量时必须转义所有特殊字符，但即便如此，你得到的变量仍然包含无法稍后解析的特殊字符，除非将它们暴露出来。解决方法是进行两层转义：

```
set uglyData=This^^^^is^^^|some%%%%messed^^^"up^^^&data
```

使用这个 set 命令，所有的双插入符号都会被解析为单插入符号。解释器会丢弃所有其他插入符号，因为它们是在转义其他字符；双百分号会解析为一个，从而使得四个变成两个。最后，解释器将 uglyData 设置为 This^^is^|some%%messed^"up^&data 文本。下一次我们解析这个变量时，现有的转义字符将被丢弃。

对 for /F 命令也做了更改：

```
for /F usebackq^ tokens^=1-6^ delims^=^"^|%%^^^& %%a in ('%uglyData%') do (
   > con echo %%a %%f %%b no longer %%d %%e.
) 
```

该命令包含了更复杂的 delims 子句，包含五个已转义的分隔符。输入到 for 命令中的 %uglyData% 文本会解析为 This^is|some%messed"up&data，且被单引号包围，并且 usebackq 关键字也已就位，并带有其后缀转义字符。

由于双引号是输入的一部分，因此此 for /F 命令必须在输入字符串周围使用单引号，这就需要使用 usebackq 关键字。（这正是 第十九章 中提到的那种罕见情况。）注意，当双引号位于正在读取的文件中时，Batch 更容易处理它；通常，解析字符串比解析文件数据要复杂。最后，我们看到 This data is no longer messed up. 被写入控制台。

从这次讨论中，最大的收获是你应该始终弄清楚需要预期多少级别的转义，并确保代码能够支持输入中所有预期的特殊字符。

### 总结

在本章中，我介绍了一些我在作为批处理编码员的职业生涯中遇到的高级技巧。这并不是关于`for`命令的所有有趣难题的详尽列表，但它代表了许多典型情况。要充分发挥批处理的能力，你必须理解`for`命令，而这需要实验和探索。不要害怕。如果你在批处理中编程足够长时间，你会遇到一些这里没有涉及的内容，但如果你遵循这些示例中的思考过程，你应该能够通过实验找到解决方案。

这部分内容结束了第二部分，但这绝不是你在接下来的章节中最后一次见到`for`命令。这个命令的更多应用还在后面，但在下一章，我会稍微退一步，探索一些重要的伪环境变量。
