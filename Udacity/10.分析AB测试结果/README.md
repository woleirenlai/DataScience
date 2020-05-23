# 关于项目10：分析A/B测试结果

## 我做了什么

* 在JupyterNotebook中对网站新旧页面A/B测试结果进行分析，回答相关概率问题、假设检验问题以及使用逻辑回归对测试结果进行分析
* 载入数据对数据进行解评估清洗成符合测试规格要求的数据集，判断各组的转化率以及用户进入新页面的概率。 
  * 对是否转化列`converted`使用了`.value_counts(True)`方法来求是否转化的比例；  
  
  * 实验组`treatment`应对应`new_page`，在求其不一致次数时分别将对应数据提取转换成布尔值，再对布尔值求和；  
  
  * 使用不一致的布尔值作为索引，使用`df[~mismatch].copy()`创建符合要求的数据集，使用`~`排除不一致数据；  
  
  * 查看冗余数据:
  
    ```python
    df2[df2.duplicated(['user_id'], keep = False)]
    ```
  
    使用索引定位删除冗余
  
    ```python
    df2.drop([2893], axis = 0, inplace = True)
    ```
  
    之后对实验结果进行假设检验，确定零假设和备择假设，使用自助抽样法模拟进行统计学检验。
  
  * 在零假设条件下，对p_new概率下抽n_new个0、1值，同理取p_old概率下抽n_old个0、1值，之后求抽样结果下新旧页面转化率之差
  
    ```python
    #新旧页面转化率模拟抽样
    new_page_converted = np.random.choice([0,1], size = n_new, p = [1- p_new, p_new])
    old_page_converted = np.random.choice([0,1], size = n_old, p = [1- p_old, p_old])
    
    #计算抽样结果转化率之差
    p_diff = new_page_converted.mean() -  old_page_converted.mean()
    ```
  
    使用这个方法模拟10000次抽取后计算新旧页面转化率之差，编写一个`for`循环完成整个过程，将10000个新旧页面转化率之差存储在一个list中，绘制这个list的直方图，
  
    ```python
    #模拟抽样取值
    p_diffs = []
    for i in range(10000):
        new_page_converted = np.random.choice([0,1], size = n_new, p = [1- p_new, p_new])
        old_page_converted = np.random.choice([0,1], size = n_old, p = [1- p_old, p_old])
        pdiffs = new_page_converted.mean() -  old_page_converted.mean()
    p_diffs.append(pdiffs)
    
    #绘制直方图    
    plt.hist(p_diffs)
    plt.axvline(x = p_diff, color = 'red');
    ```
  
    大致呈正态分布，同时零假设下的抽样新旧页面转化之差在图中。
  
  * 计算原始数据新旧页面转化率之差后求抽样结果的新旧页面转化率之差大于原始数据新旧页面转化率之差的比例，这个比例就是统计检验的p值。
  
    ```python
    #计算原数据观察值
    p_new_obs = df2.query('landing_page == "new_page"')['converted'].mean()
    p_old_obs = df2.query('landing_page == "old_page"')['converted'].mean()
    p_diffs_obs = p_new_obs - p_old_obs
    #转为array格式
    p_diffs_obs = np.array(p_diffs_obs)
    #计算比例
    (p_diffs > p_diffs_obs).mean()
    ```
  
  * P值是零假设为真的条件下，观察到统计量的概率，如果P值越大，说明观察到统计量的概率越大，也就没有足够证据拒绝零假设。
  
  * 使用`statsmodels.api`内置函数`stats.proportions_ztest`计算统计量和p值：
  
    ```python
    import statsmodels.api as sm
    
    convert_old = df2.query('landing_page == "old_page"')['converted'].sum()#旧页面转化的次数
    convert_new = df2.query('landing_page == "new_page"')['converted'].sum()
    n_old = df2.query('landing_page == "old_page"')['landing_page'].count()#旧页面访问的次数
    n_new = df2.query('landing_page == "new_page"')['landing_page'].count()
    
    sm.stats.proportions_ztest([convert_new, convert_old], [n_new, n_old], alternative='larger')
    
    #计算z统计量和p值
    z_score, p_value = sm.stats.proportions_ztest([convert_new, convert_old], [n_new, n_old], alternative='larger')
    ```
  
  * 使用`scipy.stats`中`norm`的`norm.ppf(1-(0.05))`计算95%置信水平下，单尾Z score的临界值，使用`norm.cdf(z_score)`计算`z score`的显著性，得出结论。
  
    ```python
    from scipy.stats import norm
    #95%置信水平下，单尾Z score的临界值
    norm.ppf(1-(0.05))
    
    #Z score的显著性
    norm.cdf(z_score)
    ```
* 使用逻辑回归获得A/B测试的结果。
  * 每行的值是是否转化，选用逻辑回归来对结果进行预测。
  
  * 处理数据，为每个用户收到的页面创建虚拟变量列和截距列，使用`pd.get_dummies(df2['group'])`方法让每个用户在`treatment`组时`ab_page`列记为1，在`control`组时`control`列记为1，其余记为0。
  
  * 使用`statsmodels.api`的`.Logit()`方法拟合该模型：
  
    ```python
    log_mod = sm.Logit(df2['converted'], df2[['intercept', 'ab_page']])
    ```
  
  * 对模型使用`log_mod.fit()`赋值后使用`.summary()`查看模型摘要。
    
    * 模型摘要中，`ab_page`列的p值与上述自助抽样假设检验中的p值并不相同，原因是两个问题的零假设和备择假设不同，
    
  * 在进行假设检验时，针对自助抽样的检验是旧页面的转化率是否大于或等于新页面，这是一个单尾检验(one-tail test)，P值越大，说明越无法拒绝零假设。回归模型中的假设检验的是拟合的自变量系数是否为零，这是一个双尾检验(two-tails test)，所得的P值越小，说明越没有足够证据支持零假设，即我们没有足够的证据拒绝备择假设自变量回归系数为0，回归结果具有统计显著性.
* 添加其他变量，重新进行逻辑回归，观察结果。
  * 在添加其他影响用户是否转化的因素时，需要考虑添加其他因素可能会在一定程度上更好理解模型，在一定程度上避免辛普森悖论，但是添加附加项如果构建多元线性回归模型，首先要考虑附加项的与因变量是否为非线性关系，以及异常值和多重共线性的影响。
  * 读取包含国家信息的文件，通过`user_id`关联到当前数据集，选定`CA`为baseline，使用`pd.get_dummies()`方法创建国家的虚拟变量，重新进行逻辑回归。
    * 回归结果中国家变量的p值均>0.05，说明无统计显著性，没有足够的证据拒绝自变量系数的零假设：自变量系数为0.
  * 添加页面*国家的变量，选定`CA`为baseline，再次进行逻辑回归，观察回归结果。
    * 回归结果中的拟合优度有了少量提升，但是新加入的自变量的p值仍>0.05，说明无统计显著性，无法拒绝自变量系数的零假设。
* 考虑实验时长对结果的影响，计算实验的花费时间，得出结论。



以下为官方项目概述

---

---

## 分析 A/B 测试结果

数据分析师和数据学家经常使用 A/B 测试。如何分析A/B测试结果非常重要。

在这个项目中，我们设定了一个电子商务网站运营 A/B 测试的情境。该公司设计了新网站页面，想提高用户转化率，即购买公司产品的用户数量。你的目标是通过分析来帮助公司决定他们是否应该使用新页面，还是保留原有网页或延长测试时间来进一步观测。

你所需要的数据和 Jupyter Notebook 是可供下载的压缩文件，你可以从资源标签 (以及下面的 "辅助材料") 中获取。我们也设置了 5 个测试帮助你更好地完成项目。你可以先阅读项目 Notebook，然后返回来做测试。注意，你提交的项目必须满足 [审阅规范](https://review.udacity.com/#!/rubrics/1331/view) 中的所有要求。

这一实战项目主要涵盖统计课程知识，希望你在做项目时能全面巩固所有相关知识 。 祝你好运！

## 简介

A / B 测试是数据分析师和数据科学家经常做的工作之一，非常重要。本项目模拟了一家电子商务网站运行的 A / B 测试的结果。你的目标是帮助公司分析并判断他们是否应该运行新的页面，或保留旧的页面，还是应该延长试验时间再来做决策。

在提交项目前需要检查项目是否符合[审阅标准](https://review.udacity.com/#!/rubrics/1331/view)。

------

如果你看到这个页面，那么你应该已经完成了 Jupyter Notebook 上的所有问题，准备好了一个.html文档的副本。在完成提交之前，请阅读以下几点做最终检查。 如果还没有完成项目，请返回上一页，在你的系统上设置好 Python，然后完成项目。

## 项目审阅标准

你的项目将由一位优达学城的审阅专家根据[项目审阅标准](https://review.udacity.com/#!/rubrics/1331/view)进行评估。提交前请一定仔细阅读，确保你提交的内容符合规范，这也有助于你培养良好的编码习惯。

## 收集提交材料

如果确认不再修改 notebook，你应该将其保存为容易读取的格式。你可以使用**File - > Download as - > HTML（.html）**菜单将 Notebook 保存为.html 文件。如果出现“No module name”错误，请打开终端并尝试使用`pip install ` 安装缺少的模块（在模块名称中不包括“<”or“>”或其他文字）。

请同时提交原始 Notebook 和 Notebook 的 HTML 或 PDF 副本以供审阅。你不需要在提交中包含任何数据文件。如果你参考了其他网站、书籍和其他资源来帮助你解决项目中的任务，请确保记录在项目中。建议你在 Notebook 报告末尾的 Markdown 单元格中添加“参考资源”部分，或者可以包含一个记录参考文件出处的`readme.txt`文件。

## 提交项目

准备就绪后，你可以点击“提交项目”按钮进入项目提交页面。你可以将文件作为 .zip 压缩文件进行提交，也可以链接到包含项目文件的GitHub 仓库。如果你使用 GitHub，请注意，你提交的内容将是提交时链接仓库的快照。建议你将每个项目保存在一个单独的仓库中，以避免任何混淆的可能。

项目评估一般在一周以内。一旦你提交的项目被审阅，你将收到一封电子邮件。如果你在提交项目时遇到任何问题，或者希望查看提交的状态，请发送电子邮件至 [support@youdaxue.com](mailto:support@youdaxue.com)。

中文版Notebook可以通过[这里](https://s3.amazonaws.com/video.udacity-data.com/topher/2018/April/5ae2df6a_analyze-ab-test-results-notebook-zh-0/analyze-ab-test-results-notebook-zh-0.ipynb)下载。（右键->另存为）