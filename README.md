# 电力人工智能数据竞赛自动评测

由于本次电力人工智能数据竞赛采用了基于国网电力真实工业场景的指标，因此我们给出该指标计算方法和自动评测脚本的详细介绍。

## 环境要求
* python==3.7.6
* terminaltables==3.1.0  
* torch==1.6.0
* tqdm==4.42.1

## 项目结构
```
.
├── README.md
├── crane_evaluation
│   ├── main.py                 # 吊车评测主函数
│   ├── test.json               # 真实标注吊车json文件（未公开）
│   ├── test_pred_simple.json   # 选手提交预测吊车json文件
│   └── utils                   # 评测功能函数
│       ├── options.py
│       └── utils.py
├── helmet_evaluation
│   ├── main.py                 # 安全帽评测主函数
│   ├── test.json               # 真实标注安全帽json文件（未公开）
│   ├── test_pred_simple.json   # 选手提交预测安全帽json文件
│   ├── utils                   # 评测功能函数
│   │   ├── options.py
│   │   └── utils.py
└── requirements.txt            # 需要安装的python库
```
注：由于`test.json`为测试集标注答案，因此不能公开。

## 运行方式
计算给定预测结果和真实结果的国网电力指标并且返回：
* 安全帽：  
  * 进入`helmet_evaluation`文件夹  
  * `python main.py --contestant_submitted_file_name=test_pred_simple.json`  
* 吊车：  
  * 进入`crane_evaluation`文件夹  
  * `python main.py --contestant_submitted_file_name=test_pred_simple.json`

## 实验指标 
* ~~正确率：P (precision) = TP / (TP + FP)，所有预测出来的正例中有多少是真的正例~~
* ~~召回率：R (Recall) = TP / (TP + FN)，所有真实正例中预测出了多少真实正例~~  
* ~~F1值：F1 Score = 2 * P * R / (P + R)，精确率和召回率的调和均值~~ 
* ~~mAP (mean Average Precision): 目标检测模型的评估指标~~
* 国网电力指标（已更新）  
 该指标基于国网电力真实工业应用场景设计，对于安全帽和吊车两个数据集只关注隐患目标（没戴安全帽的，有吊车的），并且以下统计指标时只考虑识别出来的隐患目标（识别非隐患目标直接过滤掉）和真实情况的隐患目标
    * 安全帽评测基准：
        * 误检率 = 不存在隐患被识别出图像数量 / 检出的图像总数量
            * 不存在隐患被识别出图像数量的含义：
                * 图像没有未戴安全帽目标（没有人或者所有人都戴了安全帽）：如果检测结果非空（不用考虑IOU >= 0.5），算是误检；如果检测结果为空，不算误检
                * 图像含有未戴安全帽目标：如果检测结果非空并且至少正确识别一个（要求IOU >= 0.5），不算误检；如果检测结果非空并且都识别错误（要求IOU >= 0.5，结果IOU < 0.5），算是误检；如果检测结果为空，不算误检
            * 检出的图像总数量的含义：
                * 检测结果含有未戴安全帽目标的图像总数（不用考虑IOU >= 0.5）
        
        * 漏检率 = 存在隐患但未识别出图像数量 / 具有通道隐患的图像总数量
            * 存在隐患但未识别出图像数量的含义：
                * 图像含有未戴安全帽目标：如果检测结果非空并且至少正确识别一个（要求IOU >= 0.5），不算漏检；如果检测结果非空并且都识别错误（要求IOU >= 0.5，结果IOU < 0.5），算是漏检；如果检测结果为空，算是漏检
            * 具有通道隐患的图像总数量的含义：
                * 真实结果含有未戴安全帽的图像总数
        
        * 识别准确率 = 目标识别正确数量 / 测试集标准目标数量
            * 目标识别正确数量的含义：
                * 检测结果非空，检测目标正确的数量（要求IOU >= 0.5）
            * 测试集标准目标数量的含义：
                * 真实结果含有未戴安全帽的目标总数

    * 吊车评测基准：
         * 误检率 = 不存在隐患被识别出图像数量 / 检出的图像总数量
            * 不存在隐患被识别出图像数量的含义：
                * 图像没有吊车目标：如果检测结果非空（不用考虑IOU >= 0.5），算是误检；如果检测结果为空，不算误检
                * 图像含有吊车目标：如果检测结果非空并且至少正确识别一个（要求IOU >= 0.5），不算误检；如果检测结果非空并且都识别错误（要求IOU >= 0.5，结果IOU < 0.5），算是误检；如果检测结果为空，不算误检
            * 检出的图像总数量的含义：
                * 检测结果含有吊车的图像总数（不用考虑IOU >= 0.5）

         * 漏检率 = 存在隐患但未识别出图像数量 / 具有通道隐患的图像总数量
            * 存在隐患但未识别出图像数量的含义：
                * 图像含有吊车目标：如果检测结果非空并且至少正确识别一个（要求IOU >= 0.5），不算漏检；如果检测结果非空并且都识别错误（要求IOU >= 0.5，结果IOU < 0.5），算是漏检；如果检测结果为空，算是漏检
            * 具有通道隐患的图像总数量的含义：
                * 真实结果含有吊车的图像总数
         
         * 识别准确率 = 目标识别正确数量 / 测试集标准目标数量
            * 目标识别正确数量的含义：
                * 检测结果非空，检测目标正确的数量（要求IOU >= 0.5）
            * 测试集标准目标数量的含义：
                * 真实结果含有吊车的目标总数

    * 分数评定：
        * 初赛均采用线上评分，以模型性能分数为准
        * 模型性能分数 = 1 - (A1 * 误检率 + A2 * 漏检率 + A3 * (1 - 识别准确率))
        * 其中A1 = 0.3，A2 = 0.5，A3 = 0.2

     * 出现以下几种情况，此条图像结果会被判为错误：
        * 预测单项目标框数量超过标准答案中框数量的2倍：
            * 安全帽：预测未戴安全帽目标框数量超过标准答案中未戴安全帽目标框数量的2倍（不用考虑IOU >= 0.5），算是误检
            * 吊车：预测吊车目标框数量超过标准答案中吊车目标框数量的2倍（不用考虑IOU >= 0.5），算是误检
       * 结果文件（名称、标签、大小写、未定义类型等等）不规范

## 实验结果
初赛安全帽测试集基于YOLOv3模型的结果：
以`helmet_evaluation/test_pred_simple.json`和`helmet_evaluation/test.json`为例，计算国网电力指标为： 
<div class="table">
<table border="1" cellspacing="0" cellpadding="10" width="100%">
<thead>
<tr class="firstHead">  
<th colspan="1" rowspan="1">false detection rate</th> <th>missed detection rate</th> <th>object detection correct rate</th> <th>sgcc helmet image score</th>
 </tr>
</thead>
<tbody>
<tr>
<td>0.6256281407035176</td>
<td>0.27626459143968873</td> <td>0.6104078762306611</td> <td>0.5962608373152326</td>
</tr>
</tbody>
</table>
</div>

初赛吊车测试集基于YOLOv3模型的结果：
以`crane_evaluation/test_pred_simple.json`和`crane_evaluation/test.json`为例，计算指标为： 
<div class="table">
<table border="1" cellspacing="0" cellpadding="10" width="100%">
<thead>
<tr class="firstHead">  
<th colspan="1" rowspan="1">false detection rate</th> <th>missed detection rate</th> <th>object detection correct rate</th> <th>sgcc crane image score</th>
 </tr>
</thead>
<tbody>
<tr>
<td>0.5656324582338902</td>
<td>0.10780669144981413</td> <td>0.7336343115124153</td> <td>0.723133779107409</td>
</tr>
</tbody>
</table>
</div>

## 选手问题答疑
问题1：
`for class_item in gt_data['categories']:`  
提交结果`pred_data`是一个`list`，那么标注结果`gt_data`应该也是一个`list`，这里当成一个`dict`来用了？  
解决方案：提交结果`pred_data`是一个`list`，标注结果`gt_data`应该是一个`dict`。

问题2：
当前评测指标的规则当中的预测单项目标框数量超过标准答案中框数量的2倍，是针对安全帽赛道还是吊车赛道的？  
解决方案：这条规则同时适用于安全帽赛道和吊车赛道，之前的评测版本由于和国网电力第一次沟通时只针对安全帽赛道，因此只有安全帽赛道的评测脚本使用了该规则，当前已经更新为安全帽赛道和吊车赛道同时使用。

问题3：
当前安全帽赛道的评测指标是按照戴安全帽计算还是未戴安全帽计算的？  
解决方案：当前安全帽赛道的评测指标的规则是按照未戴安全帽计算，之前的评测版本由于和标注公司沟通时没有确认两个类别的标签，因此计算的是戴安全帽的结果，当前已经更新为未戴安全帽的结果，线上评测系统正在更新。
