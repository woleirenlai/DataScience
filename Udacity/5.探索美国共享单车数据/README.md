# 关于项目5：探索美国共享单车数据

## 我做了什么

* 编写在Terminal里可以运行的Python程序，通过在Terminal交互输出共享单车数据的分析结果。

* 事先创建字典类型的`CITY_DATA`方便读取数据，事先定义月份初始变量以及周初始变量。

* 在获取输入的函数`get_filters()`中创建控制输入的函数`input_mod()`，输入输入项、错误值、初始值、对应值，在每次获取`city`、`month`、`day`的输入时调用，输出输入的对应数据，以减少重复代码。

* 控制输入的函数`input_mod()`使用`while`控制循环，使用`if-else`判断如果输入的获取的值不在初始的数据中则返回错误值，使用`str.format()`方法和`str.title()`方法规范输入和强调。

  ```python
  #创建一个用于控制输入的函数
  def input_mod(input_print, error_print, enterable_list, get_value):
      while True:
          ret = input(input_print)
          ret = get_value(ret) #这里执行作为参数的函数
          if ret in enterable_list:
              print('You have selected {}.'.format(ret.title()))
              return ret
          else:
              print(error_print)
  ```


* 获取`city`、`month`、`day`调用`input_mod()`，在对应已选择数据使用`lambda x: str.lower(x)`规范输出的数据格式。

* 在加载数据函数`load_data()`中，城市读取为`CITY_DATA`中key对应的value，构建一个dataframe存储，转换时间列为datetime格式，通过`df['datetime_column'].dt.month`与`dt.day_name()`方法生成月份以及周几两列，判断输入的`month`不为all时，处理`month`实际为`months`列表索引+1，同时根据已选择的`month`对dataframe筛选月份，判断输入的`day`，不为all时，根据已选择的`day`对dataframe筛选周几。

  ```python
  def load_data(city, month, day):
  
      df = pd.read_csv(CITY_DATA[city])
  
      # convert the Start Time column to datetime
      df['Start Time'] = pd.to_datetime(df['Start Time'])
  
      # add two new columns: month column and day of week column from Start Time
      df['month'] = df['Start Time'].dt.month
      df['day_of_week'] = df['Start Time'].dt.day_name()
  
      # filter by month if applicable
      if month != 'all':
          months = ['january', 'february', 'march', 'april', 'may', 'june']
          month = months.index(month) + 1
      # filter by month to create the new dataframe
          df = df[df['month'] == month]
  
      # filter by day of week if applicable
      if day != 'all':
      # filter by day of week to create the new dataframe
          df = df[df['day_of_week'] == day.title()]
  
      return df
  ```

* 在时间统计函数`time_stats()`中，使用`df.column.mode[0]`取众数判断最常见的月份、周几、起始时间，在起始时间的处理使用`df.datetime_column.dt.hour`获取时间，所有的输出打印使用`str.format()`进行规范。

* 在站点统计函数`station_stats()`中，使用`df.column.mode[0]`取众数判断最常见起始站点和结束站点以及频率最高的起始-结束旅途，在判断最高频率时将原始dataframe按照起始站和结束站分组后使用`size()`方法和`idxmax()`方法获取最大索引，所有的输出打印使用`str.format()`进行规范。

  ```python
  def station_stats(df):
      """Displays statistics on the most popular stations and trip."""
  
      print('\nCalculating The Most Popular Stations and Trip...\n')
      start_time = time.time()
      # display most commonly used start station
      most_commonly_used_start_station = df['Start Station'].mode()[0]
      print('The most commonly used start station is : {}.'.format(most_commonly_used_start_station))
  
      # display most commonly used end station
      most_commonly_used_end_station = df['End Station'].mode()[0]
      print('The most commonly used end station is : {}.'.format(most_commonly_used_end_station))
  
      # display most frequent combination of start station and end station trip
      #Modified as reviewed
  
      top = df.groupby(['Start Station', 'End Station']).size().idxmax()
      print('The most frequent combination of start station and end station trip is:\n' \
            '{} to {}'.format(top[0], top[1]))
  
      print("\nThis took %s seconds." % (time.time() - start_time))
      print('-'*40)
  ```

* 在用车时间统计函数`trip_duration_stats()`中，对用车时间列使用`.sum()`、`.mean()`、`.max()`、`.min()`求总计用时、平均用时、最长用时、最短用时，所有的输出打印使用`str.format()`进行规范。

* 在用户统计函数`user_stats()`中，观察数据集已知`washington`数据集缺少`Gender`和`Birth Year`列，使用`if-else`判断需要进行统计的列名在数据集中，则继续，不在则输出错误，用户统计对用户类型、性别使用`.value_counts()`方法进行计数，对出生年份使用`.min()`、`.max()`、`.mode()[0]`求最小值、最大值和众数，所有的输出打印使用`str.format()`进行规范。

* 主函数`main()`对整体所有输入进行控制，输出输入的分析结果，使用`while`和`break`控制是否重新开始。

  ```python
  def main():
      while True:
          city, month, day = get_filters()
          df = load_data(city, month, day)
  
          time_stats(df)
          station_stats(df)
          trip_duration_stats(df)
          user_stats(df)
  
          restart = input('\nWould you like to restart? Enter yes or no.\n')
          if restart.lower() != 'yes':
              break
  ```

以下为官方项目概述

---

---

## 概述

在此项目中，你将利用 Python 探索与以下三大美国城市的自行车共享系统相关的数据：芝加哥、纽约和华盛顿特区。你将编写代码导入数据，并通过计算描述性统计数据回答有趣的问题。你还将写一个脚本，该脚本会接受原始输入并在终端中创建交互式体验，以展现这些统计信息。

## 自行车共享数据

在过去十年内，自行车共享系统的数量不断增多，并且在全球多个城市内越来越受欢迎。自行车共享系统使用户能够按照一定的金额在短时间内租赁自行车。用户可以在 A 处借自行车，并在 B 处还车，或者他们只是想骑一下，也可以在同一地点还车。每辆自行车每天可以供多位用户使用。

由于信息技术的迅猛发展，共享系统的用户可以轻松地访问系统中的基座并解锁或还回自行车。这些技术还提供了大量数据，使我们能够探索这些自行车共享系统的使用情况。

在此项目中，你将使用 [Motivate](https://www.motivateco.com/) 提供的数据探索自行车共享使用模式，Motivate 是一家入驻美国很多大型城市的自行车共享系统。你将比较以下三座城市的系统使用情况：芝加哥、纽约市和华盛顿特区。

## 数据集

我们提供了三座城市 2017 年上半年的数据。三个数据文件都包含相同的核心**六 (6)** 列：

- 起始时间 Start Time（例如 2017-01-01 00:07:57）
- 结束时间 End Time（例如 2017-01-01 00:20:53）
- 骑行时长 Trip Duration（例如 776 秒）
- 起始车站 Start Station（例如百老汇街和巴里大道）
- 结束车站 End Station（例如塞奇威克街和北大道）
- 用户类型 User Type（订阅者 Subscriber/Registered 或客户Customer/Casual）

芝加哥和纽约市文件还包含以下两列（数据格式可以查看下面的图片）：

- 性别 Gender
- 出生年份 Birth Year

原始文件（访问地址：([芝加哥](https://www.divvybikes.com/system-data)、[纽约市](https://www.citibikenyc.com/system-data)、[华盛顿特区](https://www.capitalbikeshare.com/system-data)）有很多列，并且在很多方面格式不一样。我们执行了一些[数据整理](https://en.wikipedia.org/wiki/Data_wrangling)操作，使这些文件压缩成上述核心六大列，以便于更直接地使用 Python 技能进行评估和分析。在后续的数据分析师纳米学位（进阶）课程中，有一门课程专门讲解如何整理最杂乱无章的数据集。因此不用担心不了解这方面的知识。

## 文件

要使用 Python 回答这些问题，你需要编写 Python 脚本。为了帮助你完成此项目，我们以可下载 **.py** 文件的形式提供了一个模板，其中包含辅助代码和注释。

你还需要数据集文件，即 CSV 文件；转到下一页可以详细了解你将编写的代码和数据。



请使用 `bikeshare.py` 提供的模板完成这个项目。你可以随意按照需要更改模板。要想通过项目，你的项目必须提供模板中说明的数据，并实现与用户的互动式体验。

## 互动式体验

该文件是一个脚本，它接受原始输入在终端中创建交互式体验，来回答有关数据集的问题。这种体验之所以是交互式的，是因为根据用户输入的内容，下一页面中的数据结果也会随之改变。交互式体验可以通过 `input()` 语句来进行实现，我们已经给出了 4 个提示。（提示：如果你对本要求的实现没有头绪，可以回顾 课程5：脚本编写 -> 8. 在脚本中接受原始输入。）

有以下三个问题会对结果产生影响：

1. 你想分析哪个城市的数据？输入：芝加哥，纽约，华盛顿 ( Would you like to see data for Chicago, New York, or Washington?)
2. 你想分析几月的数据？输入：全部，一月，二月…六月 ( Which month? all, january, february, ... , june?)
3. 你想分析星期几的数据？输入：全部，星期一，星期二…星期日 (Which day? all, monday, tuesday, ... sunday?)

这几个问题的答案将用来确定进行数据分析的城市，同时选择过滤某个月份或星期的数据。在相应的数据集过滤和加载完毕后，用户会看到数据的统计结果，并选择重新开始或退出。输入的信息应当大小写不敏感，比如"Chicago", "CHICAGO", "chicago", “chiCago”都是有效输入。你可以使用 `lower()`, `upper()`, `title()`等字符串方法对输入值进行处理。

**请注意，`bikeshare.py` 文件是一个供你使用的模板，但你并不是必须使用它。** 只要你的文件拥有与模板相似的交互式体验，你可以随意更改它的功能。我们鼓励你改变 `bikeshare.py` 的结构（比如添加和/或删除辅助函数），让代码变得更加高效简洁！



## 问题

你将编写代码并回答以下关于自行车共享数据的问题：

- 起始时间（Start Time 列）中哪个月份最常见？
- 起始时间中，一周的哪一天（比如 Monday, Tuesday）最常见？ 提示：可以使用 `datetime.weekday()` （点击[查看文档](https://docs.python.org/3/library/datetime.html#datetime.date.weekday)）
- 起始时间中，一天当中哪个小时最常见？
- 总骑行时长（Trip Duration）是多久，平均骑行时长是多久？
- 哪个起始车站（Start Station）最热门，哪个结束车站（End Station）最热门？
- 哪一趟行程最热门（即，哪一个起始站点与结束站点的组合最热门）？
- 每种用户类型有多少人？
- 每种性别有多少人？
- 出生年份最早的是哪一年、最晚的是哪一年，最常见的是哪一年？

## 文件

要使用 Python 回答这些问题，你需要编写 Python 脚本。为了帮助你完成此项目，我们以可下载 **.py** 文件的形式提供了一个模板，其中包含辅助代码和注释。

你还需要数据集文件，即 CSV 文件；转到下一页可以详细了解你将编写的代码和数据。

## *TO DO*

你在对应的代码文件中（`bikeshare.py`） 中必须填写的所有代码都用以*“TO DO”*开头的注释标记。请仔细阅读该文件，大致了解脚本流程以及为了完成此项目你需要添加的代码。



## 代码流程

Python中首先执行最先出现的非函数定义和非类定义的没有缩进的代码，会从前到后执行。在 `bikeshare.py` 中，代码的执行顺序会从 `if __name__ == "__main__":`开始，执行里面的 `main()` 函数。

在这个 `main()` 函数中，是一个条件为`True` 的 `while` 循环，除非被下方的 `if` 条件语句打破，循环会一直继续下去。

```python
restart = input('\nWould you like to restart? Enter yes or no.\n') # 你是否想要重新开始？输入 yes or no。
        if restart.lower() != 'yes': # 如果输入不为 yes
            break # 跳出 while 循环，程序结束
```

接下来我们查看 `main()` 函数的其他部分，这些语句会被顺序执行。但是这些函数并没有被完整的定义，你需要根据项目中提示的 TO DO 来补全他们。

```python
# 通过用户的输入来得到要分析的 “城市，月，日”
city, month, day = get_filters() 

# 加载相应的 “城市，月，日” 的数据
df = load_data(city, month, day) 

# 计算并显示共享单车出行的最频繁时间
time_stats(df) 

# 计算并显示共享单车出行的最频繁车站
station_stats(df) 

# 计算并显示共享单车出行的总/平均时间
trip_duration_stats(df) 

# 计算并显示共享单车用户的统计信息
user_stats(df) 
```



## 交互式体验

`bikeshare.py` 文件是一个脚本，它会接受原始输入以在终端中创建交互式体验，该体验将回答关于数据集的问题。体验之所以是交互式体验，是因为根据用户的输入，上一页的问题的答案将改变！下面的四个问题将改变答案：

1. 你要查看芝加哥、纽约还是华盛顿特区的数据？
2. 你要按照月份、日期过滤数据，还是不过滤数据？
3. 如果按月份过滤，哪个月份？1 月？2 月？3 月？4 月？5 月？6 月？
4. 如果按日期过滤，哪个日期？

要完成项目，请在 [工作区]在教室内完成项目 中使用`bikeshare.py`。

正确地使用 NumPy 和/或 pandas 意味着需要使用向量化的运算，你暂时不需要理解这方面的概念。只需要明白你可以在 NumPy 和 pandas 中循环访问这些数据结构。但使用这两个库使我们很难评估在这门课程中讲授的核心概念（数据类型和运算符、控制流、函数等）。

我们我们已经在 `bikeshare.py` 提供了一些TO DO待办事项。注意这个`bikeshare.py`文件只是一个模板，你不是必须要使用它。只要你能确保最终的功能，你可以改变模板里面的函数来实现交互式体验。

交互式体验可以通过在 `bikeshare.py` 中的 `input()` 语句来进行实现，我们已经给出了 4 个提示。你需要补全 TO DO 代码以实现此交互式体验。本项目要求输入大小写不敏感，你可以使用 `lower()`, `upper()`, `title()`等字符串方法对输入值进行处理。



## 如果你遇到困难

如果你在补全这些函数的期间遇到困难，请尝试下面几种途径：

1. 查看前面的课程/练习题，是否课程中有提供的示例或文档，例如: `pandas`, `datetime`；
2. 使用搜索引擎进行搜索，找到可能实现某功能的函数/方法，再对应查看该函数/方法的文档，根据文档中提供的说明或示例，了解如何进行使用；
3. 在论坛进行提问 或者 提问你的助教。