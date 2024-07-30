# Paper Crawler for Top AI Conferences with GPT Analysis

## Conferences supported
- CVPR and ICCV since 2013.
- ECCV since 2018.
- AAAI since 1980.
- IJCAI since 2017.
- NIPS since 1987.
- ICML since 2017.
- ICLR 2018, 2019, 2021 and 2022.

## Totorial/教程（以CVPR2024目标检测为例）
### 1. 爬取所有 CVPR2024 论文的信息

(1) 环境准备（推荐使用虚拟环境）

```shell
pip install scrapy
git clone git@github.com:yzhEric/paperCrawler-GPT.git
cd paperCrawler-GPT
```

(2) 开始下载

```shell
cd conf_crawler
# 位于 paperCrawler-GPT/conf_crawler 目录下
scrapy crawl cvpr -a years=2024  -s JOBDIR=out_jobs
```

会在当前路径下生成 data.csv 文件(重命名为 cvpr2024.csv，并且已经上传)，包含所有论文的信息，一共 2715 篇论文。

### 2. 从 CSV 中提取和目标检测相关的论文

从标题和摘要中提取即可，进行初筛。为了尽可能不会将相关论文漏掉，我设置了关键词和反向关键词，请注意:反向关键词要小心设置，其中的词表示一定不关注的论文

关键词如下（在 `filter_with_keyword.py` 中）：

```python
keywords = [
    'object detection',
    'instance segmentation',
    'panoptic segmentation',
    'open-vocabulary',
    'open vocabulary',
    'open world'
]

# 反向关键词
reversed_keywords = [
    'bev',
    'active detection',
    'boundary detection',
    'anomaly',
    'oriented',
    'point cloud',
    'video instance segmentation',
    'semantic segmentation',
    'tracking',
    'video object',
    'video',
    'attribute recognition',
    '4d instance segmentation',
    'salient object detection',
    'pose estimation',
    'lidar',
    'acoustic',
    'few-shot',
    'cross-domain',
    'cross domain',
    'domain adaptive',
    'domain adaptation',
    'adaptation',
    'attacks',
    'graph generation',
    'video segmentation'
]
```

用法
```shell
python filter_with_keyword.py
```

会在当前路径下生成 filted_cvpr2024.csv 文件。 反向关键词可以根据自己的需求进行修改。经过处理，将从 2715 篇论文中筛选出了 103 篇。你可能疑惑为这么多，我大概看了下，原因如下：

1. 一些通用技术，例如提出一个新的 backbone，然后应用于目标检测，这类论文没有被删，也是合理的
2. 一些非常小众的检测方向，我没有特意设置反向关键词，因此也被保留了

### 3. 利用 ChatGPT 对论文进行相关性分析

通过构建 sys prompt 让模型对输入的论文标题和摘要进行分类，输出强相关，一般相关和无关三个类别。
在此之前，你需要在 `chatgpt_rank_papers.py` 文件中填写你使用的 API 信息（包括 API Key 和 Base url ），位置在代码中已注释出来。

填写完 API 信息后，运行
```shell
python chatgpt_rank_papers.py
```

考虑到 OpenAI 接口的不稳定，我们设置了发送请求的延时为 10s,并且如果还是失败，那么会存储对应的标注位。因此你可以将第一次运行生成的 `filted_cvpr2024.csv` 再次输入给程序，程序有类似断点重分析的功能，已经分析的会跳过，防止浪费 token。

这个 prompt 还是比较难顶的，通过实验发现很多论文都会被认为是一般相关，其实也不是说 GPT 错了，而是论文摘要写法千差万别，很难用一个 prompt 来准确的确定是否为强相关，不过肉眼来看准确率还是比较高，只不过用户需要对一般相关性论文进行手动确认。
感觉是不是要 few shot learning 一下？ 基于我们人工确认的绝对正确的摘要，然后对未知的摘要进行预测，进一步提高准确率，但是 prompt token 就会增强不少了。

考虑到 LLM 的不可靠性，你可以手动编辑和确认 `filted_cvpr2024.csv`。

### 4. 翻译摘要

为了方便人工手动编辑和确认论文，最好将其翻译为中文保存，后续比较好快速确认。

同样，在这里你也需要先像[相关性分析](#3-利用-chatgpt-对论文进行相关性分析)一样填写 AIP 信息。
```shell
python chatgpt_translation_papers.py
``` 

注意：最终上传的 `filted_cvpr2023.csv` 文件是经过手动修改了相关性参数所得。 由于整个过程都是程序自动的，因此必然会有些遗漏，也可以人工的一并补充。

### 5. 下载所有强相关一般相关论文到本地, 无关论文不下载

```shell
python download_papers.py chatgpt_filted_cvpr2024.csv
```

在确认了哪些论文是我们应该关注的后，就可以对筛选的论文进行分析了。接下来如果想全自动归纳整理，那么可以采用如下方式：

1. 借助 chatpdf 工具，自己构造问题得到答案
2. 预定几个方向，例如目标检测，实例分割等，将论文自动归类
3. 借助上述工具，对每篇文章进行创新点或者亮点整理
4. 借助上述信息，构成思维导图，方便全局预览
5. 总结 CVPR2024 目标检测方向发展趋势
6. 后续对重点论文进行慢慢精读

不过考虑到本文所总结的论文不多，手动梳理思维导图和总结其实也不用多久。

可能还是有些遗漏，欢迎大家补充。

