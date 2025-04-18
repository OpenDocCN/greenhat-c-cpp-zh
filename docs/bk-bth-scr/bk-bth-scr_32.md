

## 第二十九章：29 数组和哈希表



![](img/chapter.jpg)

“数组？Batch 没有数组！”在敢于在 Batch 代码中提到使用这种数据结构后，我收到了各种不可置信的回应。从技术角度看，这些怀疑论者确实有一定道理。数组并不是 Batch 语言的固有部分，Batch 的创建者从未预见到它们的使用。

然而，多年来，一些创新的程序员找到了构建类似数组的东西的方法，它看起来像数组，走起来像数组，甚至像数组一样嘎嘎作响（如果数组会嘎嘎作响……并且会走的话）。在本章中，我将探索在 Batch 中构建固定长度和可变长度数组的多种方法。你将学会如何遍历一个数组并访问其中的任何给定元素，以及如何初始化一个数组。我还会讨论哈希表，详细说明它们与数组的相似性和差异，然后展示如何用数据填充它们并检索数据。最重要的是，你将学到这两种工具的应用，可以在批处理文件执行期间更好地组织和存储小型和大型的相似数据集。

### 数组

无论是什么编程语言，*数组*都是一种数据结构，用来存储多个具有相同名称的变量，它们通过索引来区分。你可以将*相似变量*看作是列表中的项或元素；它们之间必须有某种共同性。一种数组可能包含你同事的名字，另一种可能包含世界各国的名称，第三个数组可能包含篮球队的成员。这些都是字符串类型变量的数组示例，但数组也可以包含其他数据类型。再举三个数组可能表示该篮球队赛季每场比赛的统计数据：得分（整数），分差（整数），或进攻效率（浮动小数）。

数组有一个独特的名称，数组中的每个元素由数组名称和一个数字或*索引*的组合表示。在大多数现代语言中，索引是从 0 开始的，这种数组被称为*零偏移数组*。相反，从 1 开始计数的数组被称为*一偏移数组*。我将只使用更常见的零偏移数组。（如果你是个固守传统的 COBOL 程序员，你也可以在 Batch 中轻松构建一偏移数组，但你可能会被年轻的同事们标记为“老古董”哦。）

上面提到的其中一个数组可能被命名为 coworker，另一个则命名为 points。points[0] 的值将是赛季第一场比赛中得分的数值，points[1] 则对应第二场比赛，以此类推。你的同事们可能没有这么有序，但 coworker[0] 会是一个人，coworker[1] 会是另一个。如果你和 100 个人一起工作，那么 coworker[99] 将是这个 100 元素数组中的最后一个元素。

在许多编程语言中，你可以在内存中定义数组，将数组的所有元素分配给特定的数据类型。正如第五章中详细描述的那样，Batch 根本不允许你将变量定义为某些数据类型，这一点在数组中也没有变化，但你可以设置以下变量：

```
set celtics[0]=Bird
set celtics[1]=McHale
set celtics[2]=Parish 
```

任何程序员都会告诉你，这看起来无疑像是名为 celtics 的数组的前 3 个元素，而它的表现正是如此。需要注意的是，在内存中没有一个统一的数据结构来包含这三个元素。相反，解释器将它们视为三个不同的普通变量，其名称恰好都以文本 celtics 开头，后面跟着一个数字和一个尾随符号。

由于 Batch 在变量名中允许的字符非常宽松，你可以轻松地将大多数键盘字符嵌入到变量名中。其他程序员可能会有不同的数组命名惯例，但我使用方括号，也叫硬括号，来表示数组索引。于是，一个数组诞生了。

#### 创建数组

你可以通过多种方式创建数组。在上一节中，我创建了一个包含三个元素的 celtics 数组。那段代码通过将三个硬编码值分配给三个硬编码变量来定义数组的三个元素。但你也可以从多个不同的来源构建固定或可变大小的数组。

##### 用户输入的固定大小数组

继续篮球主题，构建一个包含正好五个元素的数组——即首发五名球员——并通过用户从控制台输入的数据，这段代码并不复杂。固定大小的数组需要一个特定数量的赋值操作，这就需要通过 for /L 命令创建一个迭代循环（参见[第十八章）。同样，用户输入也需要使用 set /P 命令（参见第十五章）。以下是代码：

```
> con echo Please Enter the Starting Five:
for /L %%i in (0,1,4) do (
   > con set /P myTeam[%%i]=Enter Array Value %%i = &rem
) 
```

后缀&rem 仅仅是用来突出显示它前面的空格。

这个循环执行命令，要求用户输入五次，每次迭代索引%%i，从 0 到 4。Batch 编程的一个优点是，变量名可以包含其他变量，而这是许多其他语言中不容易做到的。注意，变量 myTeam[%%i]有三个组成部分：

+   以开放方括号结尾的文本字符串：myTeam[

+   %%i 解析得到的数字：从 0 到 4

+   文本中单个字符的关闭方括号: ]

在循环的第一次执行中，变量名解析为 myTeam[0]，此时代码将从控制台获取的第一个值赋给它。然后 myTeam[1]接收第二个输入值，依此类推，直到 myTeam[4]接收第五个也是最后一个输入值。在接下来的元素赋值示例中，我将使用相同的基本技巧。

##### 从参数列表创建可变大小数组

我定义了前面的数组为固定大小，但你通常无法预知数组的最终大小。以下例程使用没有选项的`for`命令（第十七章）来处理传递给它的所有参数`%*`，将每个参数添加到 parmArr（参数数组）中。

```
:BuildParmArray
 set parmArrSize=0
 for %%i in (%*) do (
    set parmArr[!parmArrSize!]=%%~i
    set /A parmArrSize += 1
 )
 set parmArr
 goto :eof 
```

这段代码并没有使用带有内建索引的迭代循环，而是针对每个参数执行一次`for`命令代码块。因此，我首先定义或初始化索引，将其定义为 parmArrSize，然后在每次循环枚举时递增它。由于我最初将其设置为 0，循环的第一次执行将第一个参数分配给 parmArr[0]。如果有 20 个参数，`for`循环会分配 20 个元素，一直到 parmArr[19]。

这是一个微妙的要点，但请注意，我在循环结束时递增 parmArrSize，设置为下一个参数的索引值，不管是否还有下一个参数。结果是，如果这段代码分配了 0 到 19 号元素，最终的 parmArrSize 值将是 20，这也是数组的实际大小。许多编程语言都有返回数组大小的方法；而在 Batch 中，最接近的做法是使用一个包含该数据项的变量。

这段代码构建了一个零偏移数组，但你可以通过稍作修改来构建一个一偏移数组。只需交换代码块中的两个`set`命令，将 parmArr[1]设置为第一个元素，parmArrSize 仍然会包含正确的数组大小。

如果调用此例程时没有传递任何参数，parmArrSize 将保持为 0，此逻辑不会向数组中添加任何内容，因为`for`命令的代码块根本不会执行。因此，这段代码成功地创建了一个空数组。

此外，我不想忽视代码块后面的`set`命令。在构建或填充数组后，我通常喜欢记录它的当前内容。此命令会将所有以 parmArr 开头的变量及其值写入标准输出（stdout）或跟踪文件中。由于数组的所有元素都以该文本开头，它们都会被显示出来。

我强烈建议对所有不是特别大的数组都做这个操作。你通常会忽略这些数据，但当需要进行故障排查时，这样做会让任务变得更加轻松。出于诊断的目的，它提供了一个很好的审计跟踪，记录了这段代码如何加载数组，而且在后续修改数组后，你可以轻松地在代码的其他地方重复这段操作。

关于这个清单的最后一点，你可能对我为索引使用的变量名有所疑问。通常最好简洁地定义索引，但 parmArrSize 这个名字的确显得有些冗长，甚至有些笨重。之所以这么命名，是因为这个变量名是数组名与 Size 文本的连接，set parmArr 命令会显示它及其值，并且同时显示数组内容。如果数组的名字相对独特，那么不太可能有其他内容满足这个条件，从而使这个命令干净地显示出你想知道的关于该数组的所有信息：它的元素及其大小。

##### 从文件中加载的共生数组

在下一个例子中，我将演示几个有趣的概念，你可以将它们一起使用，也可以单独使用。一个是从数据文件加载数组，另一个是构建共生数组。当两个数组大小相同且通过它们的索引同步时，它们就是*共生*数组。例如，一个数组包含同事，另一个数组包含电话号码，它们可能各自包含 10 个条目。这并不意味着它们是共生的，但如果索引 0 处的电话号码属于另一个数组中索引 0 处的同事，且所有 10 组元素都满足这一条件，那么这两个数组就是共生数组。

在第十五章中，我展示了一个交互式的 bat 文件，它会讲一个笑话、一个双关语或一个谜语。在第二十一章中，当讨论随机伪环境变量时，我通过想象几十个笑话、双关语和谜语，所有内容都在内存中并可以随机访问——显然给用户提供了数小时的娱乐。这个难题的最后一块拼图是读取一个包含数十个笑话的库（也就是一个文件），并将它们加载到内存中供随机访问。这听起来像是一个数组……或者可能是两个数组。

现在我们暂时只关注笑话，先将双关语和谜语放一边，知道稍后我们可以对它们做类似的处理。幸运的是，我只会包括 *BatJokes.txt* 文件中的前三行（不要与 *BadJokes.txt* 文件混淆），但请想象成有数百行内容。每条记录包含一个笑话，后面跟着它的答案，通过管道符分隔：

```
Why are bats so active at night?|They charge their bat-teries by day.
How do bats flirt?|They bat their eyes.
What's a good pick-up line for a bat?|Let's hang. 
```

清单 29-1 中的代码将这些喜剧金句加载到两个数组中，笑话加载到笑话数组，答案加载到答案数组。

```
set jokes=0
for /F "tokens=1-2 delims=|" %%b in (C:\Batch\BatJokes.txt) do (
   set joke[!jokes!]=%%~b
   set answer[!jokes!]=%%~c
   set /A jokes += 1
)
set joke
set answer 
```

清单 29-1：从数据文件构建的共生数组

这些是共生数组，因为位于特定索引处的笑话对应于相同索引处的答案。当 for /F 命令读取每条记录时，它会根据管道符分隔来标记笑话和答案的文本。前两个 set 命令使用复数形式的笑话索引将每个字符串分配给适当数组的元素。我为两个数组都使用这个索引，因为它们是共生数组，并且在代码块的末尾递增它。

循环外的 set 命令会将以下内容写入控制台，验证两个数组都已成功加载：

```
C:\Batch>set joke 
jokes=3
joke[0]=Why are bats so active at night?
joke[1]=How do bats flirt?
joke[2]=What's a good pick-up line for a bat?

C:\Batch>set answer 
answer[0]=They charge their bat-teries by day.
answer[1]=They bat their eyes.
answer[2]=Let's hang. 
```

这也验证了 3 是笑话的总数。

我已经从一个文件中构建了这两个数组，answer[1]的内容是 joke[1]内容的关键。如果这个文件有一千个条目，那么这两个共生数组将有一千个元素，而且它们的所有元素都会同步。我可以再为谜语设置两个数组，不过我必须使用不同于 answer 的名称来命名谜语答案数组。双关语没有答案，因此我可以将每一条记录都加载到一个双关语数组中。

现在我们几乎拥有了构建一个真正可用的用户界面的所有必要内容，包括蝙蝠笑话、双关语和谜语。我们可以构建幽默的库，并将这些库加载到内存中作为数组。我们可以从用户那里接收到一个请求，要求选择某种类型的幽默，并且我们可以随机决定选择哪个笑话、双关语或谜语。最后一步是能够访问这些数组——也就是提取一个笑话及其答案并显示出来。

#### 访问数组元素

为了让这个成为一个*真正的*数组，我们必须能够遍历它，重新分配元素，将元素分配给其他变量，解析元素等。为了演示如何访问数组元素，我将首先为 myChar 数组分配 14 个元素：

```
set myChar[0]=B
set myChar[1]=a
set myChar[2]=t
set myChar[3]=c
set myChar[4]=h
set myChar[5]= &
set myChar[6]=i
set myChar[7]=s
set myChar[8]=%myChar[5]%
set myChar[9]=C
set myChar[10]=o
set myChar[11]=!myChar[10]!
set myChar[12]=l
set myChar[13]=. 
```

我正在为每个元素分配一个单一字符。大多数是直接的，但有几个稍微有点有趣。第六条 set 命令将元素 5 分配为空格，并以命令分隔符结束，这样就显而易见它不是被设置为 null 或多个空格。元素 8 和元素 11 则取用了之前定义的元素值，因此元素 8 是一个空格，元素 10 和元素 11 都是字母 o。

使用硬编码索引的两种元素解析方式表明，这两种分隔符都同样有效。使用百分号时，%myChar[5]%解析为第 6 个元素，使用感叹号时，!myChar[10]!解析为第 11 个元素。当索引是一个变量时，语法更加有限，稍后你会看到。

现在我们可以编写一个 for /L 命令，它将遍历这个数组，将所有值连接成一个句子：

```
set mySentence=&
for /L %%i in (0,1,13) do (
   set mySentence=!mySentence!!myChar[%%i]!
)
> con echo %mySentence% 
```

构建完字符串后，最后一条命令会将“Batch is Cool.”写入控制台。

回到前面，for 命令将%%i 从 0 迭代到 13，其中!myChar[%%i]!依次解析为 14 个元素中的每一个。解释器首先解析%%i 索引，然后解析数组元素。例如，第一次通过循环时，中间结果是 myChar[0]。由于它被感叹号包围，解释器将其解析为 B，并将其赋值给 mySentence。第二次通过循环时，myChar[1]解析为 a，解释器将其与前一个结果连接起来，得到 Ba。这个过程重复了十几次，直到我们构建出完整的句子。

这是延迟展开功能的又一个例子，使用延迟展开时，必须使用感叹号作为外部分隔符；百分号是无法工作的。虽然很容易尝试使用%myChar[%%i]%，但是如果你像解释器那样思考，你会发现并没有数组。你会看到两个独立的变量需要解决：myChar[和 i]。在这个例子中，for 变量是索引，但延迟展开对使用普通变量作为索引时也适用。考虑以下例子，其中百分号包含 idx 索引，而感叹号包含外部数组元素，适用于第二级延迟展开：

```
set idx=9
> con echo The Tenth Element is: !myChar[%idx%]! 
```

这会将大写字母 C 写入控制台。

同样，要从 Listing 29-1 中构建的共生数组中提取笑话及其答案，你可以将一个小于笑话数量的随机非负数输入到一个索引变量中，例如 jokeIdx。然后，你可以通过!joke[%jokeIdx%]!来解析笑话。因为这些是共生数组，你可以使用相同的索引来提取答案：!answer[%jokeIdx%]!...你现在拥有了更新第十五章中的 bat 文件所需的所有工具，可以随机显示多个笑话、双关语和谜语中的任意一个。

#### 初始化数组

支持数组的语言通常提供一个简单的一行命令来创建数组，甚至重新初始化现有数组的所有元素。Batch 中没有变量的实例化，更不用说数组了，因此再次需要发挥创造力。

在构建笑话数组之前，最好先初始化它，就像我在进入循环之前常常初始化变量为 0 一样。虽然很不可能任何活动的变量以 joke[text 开头，但确实有些情况下你会想要重新初始化并重建已在使用的数组。以下命令会清空笑话数组中的所有元素：

```
for /F "usebackq delims==" %%j in (`set joke[`)  do (set %%j=)
```

for /F 命令接受输入的 set 命令，列出了数组的每个现有元素及其值。通过在等号上进行分隔，并仅传递第一个标记——即变量或元素名称，而不是其值——我们在代码块中将每个元素设置为 null。如果没有变量需要重置，第二个 set 命令就不会执行，最终结果也是一样的。

在之前的示例中，我假设在构建数组时没有任何元素已存在。为了确保准确，最好在构建和使用数组之前执行类似这样的命令来初始化数组。

#### 实现多维数组

这个技巧甚至可以扩展到构建和访问多维数组。Batch 中基本上可以使用一维数组，因为你可以在变量名中嵌入方括号。对于多维数组，我们只需要再加一个字符：逗号。以下命令将 my2dArray 数组的第 1 行、第 2 列设置为硬编码值，明显类似于二维数组元素赋值，无论使用什么语言：

```
set my2dArray[1,2]=Row1Col2Data
```

作为演示，我将简要地将之前的两个示例重新构想为二维数组：

**用户输入**

返回到由用户输入构建的数组，以下嵌套的 for /L 命令构建一个三行四列的二维数组：

```
> con echo Please Enter 2-Dimensional Array Data for 3 rows and 4 columns:
for /L %%r in (0,1,2) do (
   for /L %%c in (0,1,3) do (
      > con set /P my2dArray[%%r,%%c]=Enter Row %%r, Column %%c = &rem
)  ) 
```

外层 for 变量 %%r 遍历三行，而 %%c 则遍历每行的四列。此代码在继续之前会接受恰好 12 个值。

如果 rowIdx 设置为 2，colIdx 设置为 3，那么以下内容将解析为用户输入的最后一个数据元素：

```
!my2dArray[%rowIdx%,%colIdx%]! 
```

这两个索引首先通过百分号解析，结果为!my2dArray[2,3]!。然后，感叹号和延迟扩展完成了整个过程。

**文件输入**

之前，我们从每条输入记录构建了两个共生数组，一个用于笑话，一个用于答案。相反，我们可以将数据加载到一个单一的二维数组中，其中第二级索引 0 是笑话，1 是答案：

```
set jokes=0
for /F "tokens=1-2 delims=|" %%b in (C:\Batch\BatJokes.txt) do (
   set joke[!jokes!,0]=%%~b
   set joke[!jokes!,1]=%%~c
   set /A jokes += 1
)
set joke 
```

你可以把这个数组看作有两列；注意在两个赋值中第二维的硬编码索引：0 和 1。set /A 命令会递增笑话的索引。行数由输入文件中的记录数决定。最后，列表结尾的单一 set 命令在使用相同的三条记录输入文件时生成如下审计追踪：

```
jokes=3
joke[0,0]=Why are bats so active at night?
joke[0,1]=They charge their bat-teries by day.
joke[1,0]=How do bats flirt?
joke[1,1]=They bat their eyes.
joke[2,0]=What's a good pick-up line for a bat?
joke[2,1]=Let's hang. 
```

更大维度的数组仅仅差几条逗号：my4dArray[1,2,3,4]。我不记得曾经编写过类似的代码，但一维数组甚至二维数组在批处理中的应用非常广泛。

无论维度如何，如果你需要一个极其庞大的数组，或者如果你计划访问它成千上万次，编译代码是一个更高效的解决方案。但当合理使用时，批处理数组非常有用，且出乎意料地易于管理。

### 哈希表

事实证明，批处理确实支持数组，或者至少我们可以利用我们手头的基本工具构建一个数组。但哈希表肯定不可能吧？你可能已经从本章的标题和你现在阅读的这一节中猜到了答案，但在构建哈希表之前，让我们先定义一下它是什么。

*哈希表*（有时称为*哈希映射*）是一种以键值对存储数据的数据结构。*值*可以是任何数据类型，甚至是其他数据结构。*键*类似于数组的索引，但它不必是整数。事实上，在批处理环境中，数组和哈希表的行为非常相似；实际上，比较和对比它们的最佳方法是将数组转换为哈希表。

#### 数组与哈希表

想象一个数组，其中索引是按 CCYYMMDD 格式排列的日期，数据则是一个人一天可能走的步数。如果这个本来活跃的人在第一次大流行圣诞节期间保持静止，除了孤独地打开礼物和喝蛋酒，步数数组可能包含以下三条记录：

```
set steps[20201224]=15842
set steps[20201225]=987
set steps[20201226]=13009 
```

这些索引非常大，即使我们只使用了少量元素。幸运的是，我们没有像在许多其他语言中那样在内存中定义这个数组，因为它的大小可能会超过 2000 万个元素。Batch 数组的一个优点是，它只为已定义的元素使用内存。虽然内存已经变得相当便宜，但仍然没有理由浪费。

内存的最小使用量直接源于数组元素（例如，steps[20201225]）是一个简单的变量，变量名由一些文本、一个数字和几个方括号组成。Batch 元素的另一个优点是，索引根本不需要是数字；实际上，它甚至不需要是真正的索引。为了演示，我们将日期格式化为 MM/DD/CCYY 以提高可读性：

```
set steps[12/24/2020]=15842
set steps[12/25/2020]=987
set steps[12/26/2020]=13009 
```

由于索引不是整数，这不再是一个数组；我们已经将它转变成了哈希表。就这么简单。我们不能像遍历数组那样迭代它，但可以根据格式化后的日期查找步骤数。我们不能再通过给定的索引来检索元素；相反，查找过程涉及方括号之间的键。

进一步讲，我们可以向键中添加星期几的文本，甚至嵌入空格：

```
set steps[Thu 12/24/2020]=15842
set steps[Fri 12/25/2020]=987
set steps[Sat 12/26/2020]=13009 
```

无意间，这个关键正开始看起来像我们可以从日期伪环境变量中解析出的内容。

这个语法有一个小问题，它看起来像是一个数组。非数字键代替了索引，可能会让它看起来不像数组，但这个键很可能是一个变量，从而模糊了它的数据类型。因此，采用另一种约定是理想的。就个人而言，我选择了使用大括号，也叫括号：

```
set steps{Thu 12/24/2020}=15842
set steps{Fri 12/25/2020}=987
set steps{Sat 12/26/2020}=13009 
```

这只是其中一种约定，其他约定也同样有效。例如，你可以在哈希表变量前加上 ht。重要的是，你要使数据结构看起来像是某种独特的、不同于数组的东西。任何阅读代码的人，即使是快速浏览，也应该能注意到区别。即使读者不了解这种约定，语法的独特性也会引起好奇心，从而打开理解的大门。

这是一个简单的命令，同时又不那么简单的 set 命令：

```
set steps{%date%}=987
```

该命令正在设置步骤哈希表的一个元素，键是命令执行时的格式化日期。

如果你选择使用我的约定，我会将其总结为 array[index] 和 hashTable{key}，但像往常一样，使用适合你的约定并坚持下去。

#### 基本哈希表功能

另一个简单的哈希表示例包含了多对人员和其职业，其中人员的姓名为键，职业为值。例如，Darwin 键检索到“Naturalist”值。Lincoln、Poe 和 Braille 是总统、诗人和发明家的键。下面是该哈希表的表格形式：

| Key | Value |
| --- | --- |
| Lincoln | President |
| Darwin | Naturalist |
| Poe | Poet |
| Braille | Inventor |

注意元素缺乏索引以及任何排序的痕迹。

每个键必须是唯一值。如果这个哈希表中有两个人名为 Darwin，我们需要通过某种方式来区分他们，或许可以通过添加名字和中间名来区分。不过，多个不同的人可以拥有相同的职业，比如“诗人”；也就是说，值不必是唯一的。

在 Batch 中构建和访问哈希表元素的语法与许多其他语言非常不同，在其他语言中，在声明哈希表之后，你可以通过某种变体的*put*方法定义单个键值对。例如，这是在 Java 中实现的方式：

```
jobs.put("Lincoln", "President");
```

在 Batch 中没有点符号或内置方法，但以下的 set 命令将相同的硬编码键值对赋给哈希表：

```
set jobs{Lincoln}=President
```

更典型的是，键和值都是变量。以下代码创建了相同的条目：

```
set person=Lincoln
set aJob=President
set jobs{%person%}=%aJob% 
```

在许多语言中，提取值需要某种变体的*get*方法。这是来自 Java 的例子：

```
String job = jobs.get("Lincoln");
```

对应的 Batch 代码检索元素的方式与我们从 Batch 数组中获取元素的方式类似，只是使用了大括号和键，而不是方括号和索引。这两个命令的效果一样好：

```
> con echo The Job is: !jobs{Lincoln}!
> con echo The Job is: %jobs{Lincoln}% 
```

但键通常是变量，这就需要现在大家熟悉的延迟展开：

```
> con echo The Job is: !jobs{%person%}!
```

如果 person 设置为 Braille，输出为：The Job is: Inventor。

现在我们有了一个简单的哈希表，可以向其中添加更多的键（人名）和值（职业）作为一对一对的条目。如果某人被裁员并重新考虑自己的职业，我们也可以移除其中一条特定的记录：

```
set jobs{Jack}=&
```

以下命令仅在哈希表中尚未存在该键时，才为其分配一个值：

```
if not defined jobs{Kai}  set jobs{Kai}=Chef
```

这个 for /F 命令模拟了许多其他语言中用于获取哈希表大小以及提取键值对列表的三种不同方法：

```
set hashSize=0
set listKeys=&
set listValues=&
for /F "usebackq tokens=2-3 delims={}=" %%x in (`set jobs{`) do (
   set /A hashSize += 1
   set listKeys=!listKeys! "%%x"
   set listValues=!listValues! "%%y"
) 
```

两个列表都用空格分隔，每个元素都包含在双引号中。稍作处理，我们就可以利用这些结果来判断哈希表是否为空：

```
if not defined listKeys  > con echo The jobs hash table is EMPTY.
```

以下代码告诉我们某个特定的键是否存在于哈希表中：

```
echo %listKeys% | findstr "Poe" && > con echo Quoth the Raven, "Nevermore."
```

如果"listKeys"中包含"Poe"，则 echo 命令会将著名的副歌输出到控制台。Poe 被双引号包围，因为我们就是这样将每个键添加到哈希表中的，我们也可以类似地在 listValues 中搜索特定的值。

#### 复杂的哈希表

一个更复杂的哈希表也可能以人名为键，但它的值可能不仅仅是一个简单的字符串，而是更像现代语言中的对象。例如，它可能包含关于该人的更完整的信息，如职业、州、爱好等。（关于对象的话题，我将在第三十二章中详细讨论。）

一旦你理解了 Batch 中的哈希表和数组是通过简单地将文本、部分变量名、解析后的变量、索引和括号串联起来构建的，那么要创建一个以人名为键的人员哈希表，并将职位和州作为标识符，甚至是一个爱好数组，就不难了：

```
set people{Lincoln.job}=President
set people{Lincoln.state}=Illinois
set people{Lincoln.hobbies[0]}=Wrestling
set people{Lincoln.hobbies[1]}=Cats
set people{Lincoln.hobbies[2]}=Storytelling 
```

不要将其与点符号表示法混淆；people{Lincoln.hobbies[1]}仅仅是一个变量名，取决于你的视角，它可以是一个杂乱的名称，也可以是一个优雅的名称。

在之前的命令执行后，以下代码从哈希表中的数组提取第二个爱好：

```
set info=hobbies[1]
set person=Lincoln
set hobby=!people{%person%.%info%}! 
```

延迟扩展首先解析被百分号包围的两个变量。然后，感叹号解析中间结果为 Cats。（美国第 16 任总统与 Tabby 和 Dixie 一起住在白宫。）

我之前提到的关于数组的考虑同样适用于哈希表。这个技巧并不是为了进行重型处理，但在许多情况下，哈希表的快速且相对简单的解决方案可能就在眼前。

在第三章中，我介绍了使用延迟扩展将传输路径存储并从包含城市缩写的变量中检索的概念，比如 pathNYC、pathNash 和 pathSTL。这实际上是一个伪装的哈希表；考虑这三个元素：path{NYC}、path{Nash}和 path{STL}。将城市视为一个变量，并使用它通过!path{%city%}!来提取相应的传输路径。

### 总结

通过一些巧妙的构思，我们可以让 Batch 完成许多最初并未设计的任务，就像用旧轮胎和绳子做秋千一样。在本章中，我演示了如何从硬编码数据、用户数据、参数和文件数据加载数组。你学习了延迟扩展在访问数组元素中的重要性以及多维数组所带来的可能性。接着，我扩展了这一技巧，构建并访问哈希表，解释了它们与数组的相似性和差异，并展示了两者的应用。

我希望这些例子能够展示 Batch 中可能实现的灵活性。在下一章，我将切换话题，介绍一些虽然看似不同但对语言本身非常有用的主题。
