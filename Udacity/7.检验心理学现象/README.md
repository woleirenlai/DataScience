# 关于项目7：检验心理学现象

## 我做了什么

* 在JupyterNotebook中完成对心理学现象Stroop（斯特鲁普）实验数据集进行统计学检验的报告，Stroop实验参与者的任务是读出文字的打印颜色，数据集中记录了每个参与者读出显示的文字与打印颜色匹配与不匹配的用时。

* 确定实验的因变量和自变量，确定假设集，假设集中的零假设和对立假设。

* 确定假设检验的方法为配对样本T双尾检验，理由是数据集样本容量<30切只有样本数据，选用T检验，样本数据是同一对象的不同条件，选用配对样本T检验，对立假设中选择了总体均值差不为0，选择双尾检验。

* 对样本数据集使用`.describe()`方法进行统计描述，观察数据的集中趋势和离散趋势。

* 在一张图中绘制同一测试者在两种条件下的数据条形图，观察数据后创建时间差`diff`列绘制直方图查看分布。

  ```python
  import matplotlib.pyplot as plt
  import seaborn as sns
  %matplotlib inline
  ind = df.index  
  width = 0.35
  
  plt.figure(figsize = (12, 6))
  p1 = plt.bar(ind - width/ 2, df.Congruent, width, facecolor = '#FFB11B') 
  p2 = plt.bar(ind + width/ 2, df.Incongruent, width, facecolor = '#3A8FB7')
  
  plt.xlabel('Testers', fontsize = 12)
  plt.ylabel('Times', fontsize = 12)
  plt.title('Congruent and Incongruent test time used', fontsize = 12)
  plt.xticks(ind)
  plt.legend((p1[0], p2[0]), ('Congruent', 'Incongruent'));
  ```

  ```python
  #创建差值列
  df['diff'] = df.Incongruent - df.Congruent
  #查看分布情况
  sns.distplot(df['diff'])
  plt.xlabel('diff', fontsize = 12)
  plt.title('difference distplot', fontsize = 12);
  ```

* 导入scipy的stats库，使用`stats.ttest_rel()`方法计算两组数据的t检验下的t值和p值，查询t检验表进行统计学检验，得出结论。

  ```python
  #计算关键统计值
  from scipy import stats
  t, p_value = stats.ttest_rel(df.Congruent, df.Incongruent)
  t, p_value
  ```



以下为官方项目概述

---

---

## 项目概述

在此项目中，你将研究一个经典的实验心理学现象，叫做[斯特鲁普效应](https://en.wikipedia.org/wiki/Stroop_effect)。你将稍微了解该实验，创建关于任务结果的假设，并自己研究该任务。然后查看同样执行了该任务的其他人收集的数据，并计算描述结果的统计学数据。最后，你将用假设来解释你的结果。

该项目需要用到 T 检验的知识，作为数据分析（进阶）的先修要求，我们假设你已经具备。如果你需要再次回顾这方面的内容，可以学习我们的[这门课程](https://classroom.udacity.com/courses/ud201)。

### 为何要完成此项目？

统计学是数据分析的重要组成部分，使你能够调查数据并根据观察的结果做出推断。掌握统计学基础还能看懂其他人执行的分析，并明白他人从调查中得出的结论。

### 我将学到什么？

此项目将涉及统计学的基本概念，包括：

- 如何识别实验的组成部分
- 如何使用描述统计学描述样本数量
- 如何设置假设检验、通过样本做出推理，并根据结果得出结论

### 为何该项目对我的职业发展来说很重要？

使用统计学得出关于数据的有效结论是数据分析师的重要工作内容。熟练掌握统计学对于该纳米学位课程的后续部分来说也很必要。

## 项目说明

请遵守[这些说明](https://github.com/udacity/new-dand-advanced-china/blob/master/检验心理学现象/统计学：检验心理学现象.md)，并用 PDF 或 HTML 文档回答问题。这些文档格式与各种计算机和浏览器兼容，并能确保满足你的意图。如果你使用的是 Microsoft Word 或 LibreOffice 等 word 处理程序，确保将文档另存为 PDF 并在提交项目时包含该 PDF 文档。

对于该项目，不需要安装特定的软件，但是建议你使用 Jupyter Notebook 和 Python 完成该项目。若你对 Jupyter Notebook 还不熟悉，可以学习 [这门课程](https://classroom.udacity.com/nanodegrees/nd002-cn-advanced/parts/25558a2b-eb7f-4e2d-9047-9220a5ccd155/modules/e95ca0b1-2716-4f45-bf4b-781653e885e5/lessons/c6a12f2e-63f2-4007-a2c3-dd3e5f06f3cb/concepts/4cdc5a26-1e54-4a69-8eb4-f15e37aaab7b)。