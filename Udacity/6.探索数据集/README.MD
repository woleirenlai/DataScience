# 关于项目6：探索数据集

## 我做了什么

* 选择TMDb数据集，在JupyterNotebook中进行数据整理、探索性数据分析，生成可视化图表得出结论，形成完整数据分析报告。

* 开始导入pandas、numpy、matplotlib.pyplot包，使用`plt.rcParams['font.sans-serif'] = ['SimHei']`设置字体。

* 数据整理部分对数据理解和评估提出需要进行探索的问题，之后进行数据清理：

  * 列处理：使用`.drop([columnname], axis = 1)`方法删除与数据分析无关的列。
  * 冗余数据：使用`sum(dataframe.duplicated())`方法查看冗余数据，使用`dataframe.drop_duplicates(inplace = True)`方法删除冗余数据。
  * 空值处理：观察空值的数据量后，使用`dataframe.dropna(axis = 0, inplace = True)`方法删除空值。

* 探索性数据分析部分，根据提出的两个问题分别进行探索

  * 问题1分析电影类别

    * 处理电影类别`genres`列，原始数据由“|”将多个类别合并为一列，创建一个`set`类型数据集存储`genres`列中出现的所有类别，使用`for`循环读取`genres`列使用`str.split('|')`方法分割的`genre`，使用`set().union()`方法取每次读取的`genre`之间的并集`all_genres`，并将`all_genres`存储为`list`类型。

      ```python
      #创建一个set()，使用for循环，使用union(联合)取并集set(a).union(b)
      all_genres = set()
      for genre in df2['genres'].str.split('|'):
          all_genres = set().union(genre, all_genres)
      #将all_genres转换为列表
      all_genres = list(all_genres)
      ```

    * 使用`for`循环获取每个电影类别，对于在已处理的所有电影类别`all_genres`列表中的`genre`，原数据`genres`列包含此类别则该类别列记为1否则为0，处理完后删除`genres`列。

      ```python
      #使用for循环，获取每个电影的类型，该电影包含哪个类型，就返回1，否则返回0
      for genre in all_genres:
          df2[genre] = df2['genres'].str.contains(genre).apply(lambda x: 1 if x else 0)
      ```

    * 各类别的总数：使用`dateframe.loc[:, all_genres].sum().sort_values(ascending = True)`方法获取各个电影类别总数的降序排列，使用`.plot.barh(label = 'all_genres')`绘制横向条形图查看各个电影类别的数量。

    * 各类别的占比：使用每个电影类别/总数求占比，使用`.plot.pie()`方法绘制饼图查看占比，设置参数`explode = (proportion > 0.06)`为占比>0.06时突出显示。

    * 各类别的欢迎程度：以`all_genresd`为索引创建一个新的`dateframe`存储各个类别的受欢迎程度`popularity`的均值，绘制横向条形图查看各类别情况。

      * 对于每个类别的`popularity`，建立一个列表`list`，储存该类别对应两种情况0,1的`popularity`的均值，使用`for`循环遍历所有类别，使用`list.append()`方法在列表末尾添加对应值0,1的`popularity`均值（按`genre`分组后对`popularity`求均值），之后使用`for`循环遍历这个包含每个类别`popularity`均值的`list`，将每个类别值为1的均值存储为新的list后添加至这个新的`dateframe`中。

        ```python
        #创建新的DataFrame，以各个电影类别作为索引，列数据为该类型popularity均值
        df_genres_pop = pd.DataFrame(index = all_genres)
        #求每种类别的平均受欢迎程度
        #建立一个列表list，储存该类别对应两种情况0,1的popularity的均值
        #使用for循环遍历所有类别，使用list.append()方法在列表末尾添加对应值0,1的popularity均值
        list = []
        for genre in all_genres:
            list.append(df_genres.groupby(genre)['popularity'].mean())
        #创建一个列表，使用range(len(list1))遍历list，取list中每个类别值为1的popularity均值
        list2=[]
        for i in range(len(all_genres)):
            list2.append(list[i][1])
        df_genres_pop['popularity_mean'] = list2
        ```
      

  * 问题2分析票房高的电影特点

    * 选取收入大于平均收入的数据作为高票房数据，对这些数据使用`.describe()`方法进行描述统计。
    * 使用`dateframe.hist()`方法绘制高票房数据集的popularity，runtime，vote_average，budget_adj，revenue_adj列直方图查看其分布。
    * 使用`plt.scatter()`方法绘制popularity，runtime，vote_average，budget_adj与revenue_adj的散点图，查看他们之间的相关关系。

* 根据探索性分析的结果得出两个问题的结论，并对整个分析进行总结。





以下为官方项目概述

---

---

## 项目概述

在本项目中，你将分析一个数据集，然后传达你的发现。你将使用 Python 库 NumPy、Pandas 和 Matplotlib 来使你的分析工作更加轻松。

## 我需要安装什么？

你需要安装 Python 和以下库：

- pandas
- numpy
- matplotlib
- csv

我们建议安装 [Anaconda](https://www.continuum.io/downloads)，它自带所有必要的包和 IPython 笔记本。你可以在[此处](https://classroom.udacity.com/nanodegrees/nd002-cn-basic/parts/91b5b867-4a7f-49c5-b658-57521a8de12d/modules/e95ca0b1-2716-4f45-bf4b-781653e885e5/lessons/c6a12f2e-63f2-4007-a2c3-dd3e5f06f3cb/concepts/4cdc5a26-1e54-4a69-8eb4-f15e37aaab7b)找到安装说明。

## 为何要进行此项目？

此项目将向你介绍数据分析过程，以及所有工作的相互配合。后期的纳米学位将重点关注数据分析过程的各个环节。

你将使用 Python 库 NumPy、Pandas 和 Matplotlib，这些库使你能够更加轻松地在 Python 中编写数据分析代码！不仅如此，它们也是雇主们青睐的技能！

## 我将学到什么？

完成此项目后，你将：

*了解典型数据分析过程中所涉及到的所有步骤* 轻松提出可用给定数据集回答的问题，并解答这些问题。

- 了解如何调查数据集中的问题，以及将数据整理成你可以使用的格式
- 练习传达你的分析结果
- 能够在 NumPy 和 Pandas 中使用向量化运算，以加快数据分析代码的运行速度
- 熟悉 Pandas 的 Series 和 DataFrame 对象，它们能使你访问数据更方便
- 了解如何使用 Matplotlib 生成图形，展示你的发现



## 我要如何完成该项目？

该项目与数据分析入门课程相关，但根据你的知识背景，你可能无需学习全部课程即可完成该项目。

### 介绍

在此项目中，你需要进行数据分析，并创建一个文档分享你的发现。你应该首先了解数据集，想想它可以用来回答哪些问题。然后，你应该使用 Pandas 和 NumPy 回答你最感兴趣的问题，并编写一份报告来分享你的结论。你不需要使用推论统计或机器学习知识来完成此项目，但是你需要在报告中声明，你的结论是暂时的，可能需要进一步改进。这个项目是开放性的，没有标准答案。

### 第一步 - 选择数据集

点击[此链接](https://github.com/udacity/new-dand-basic-china/blob/master/数据分析入门/项目-探索数据集/探索数据集 - 备选数据集.md) 打开一个文档，其中包含你可以为此项目调查的数据集的链接和信息。你**必须**选择其中一个数据集来完成项目。 若数据无法下载，可以通过以下链接进行下载：

1. [TMDb电影数据](https://s3.cn-north-1.amazonaws.com.cn/static-documents/nd101/explore+dataset/tmdb-movies.csv)
2. [未前往就诊的挂号预约](https://s3.cn-north-1.amazonaws.com.cn/static-documents/nd101/explore+dataset/noshowappointments-kagglev2-may-2016.csv)
3. [Gapminder World](https://www.gapminder.org/data/)
4. [FBI 枪支数据](https://s3.cn-north-1.amazonaws.com.cn/static-documents/nd101/explore+dataset/ncis-and-census-data.zip)

### 第二步 - 组织

最终，你需要提交一份报告（并与你的朋友、家人或同事分享）。在正式开始之前，让我们进行组织整理。我们建议你新建一个文件夹，它最早将包含：

- 用来传达你的发现的**报告**文档
- 你用来分析数据的任何 **Python 代码**
- 你使用的**数据集**（你不需要提交它）

你可以使用 Jupyter Notebook，这样，你可以在同一份文档中提交编写的代码和报告你的发现。不然你可能只能分别提交报告和代码了。如果你需要一个**笔记本模板**来帮助组织你的调查，可以在页面底部找到资源链接或可以点击[此处](https://s3.amazonaws.com/video.udacity-data.com/topher/2018/August/5b7e51b8_investigate-a-dataset-zh/investigate-a-dataset-zh.ipynb)。

### 第三步 - 分析数据

思考你可以用数据集回答哪些问题，然后开始着手回答这些问题。你可以在[数据集选项](https://github.com/udacity/new-dand-basic-china/blob/master/数据分析入门/项目-探索数据集/探索数据集 - 备选数据集.md) 中找到一些问题，以帮助你入手。 试着提出那些调查多个变量之间的关系的问题。你应该在你的调查里分析至少一个因变量和三个自变量。在恰当的地方使用 NumPy 和 Pandas。

### 第四步 - 分享发现

分析完数据以后，编写一份报告，分享你觉得最有意思的发现。如果你使用 Jupyter Notebook，则与你用于执行分析的代码一起分享发现。确保你的报告文本包含在 Markdown 单元中，以清楚地将你的注释和发现与代码分开。当然你也可以使用其他工具和软件来制作最终报告，但请确保你能以 HTML 或 PDF 文件的形式提交报告，以便轻松打开。

### 第五步 - 检查

使用[**项目评估准则**](https://review.udacity.com/#!/rubrics/306/view) 来检查你的项目。如果你的提交已达到所有要求，则可以提交项目。如果你发现还有改进的空间，则继续改进吧！