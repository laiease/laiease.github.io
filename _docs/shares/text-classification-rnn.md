---
title:  中文文本分类任务
permalink: /docs/text-classification-rnn/
---

#### 环境依赖

- python >= 3.6
- paddlepaddle >= 2.2.0
- paddlenlp >= 2.2.5

#### 步骤

1. 下载PaddleNLP的源码
2. 自定义数据集
3. 修改PaddleNLP的示例代码进行模型训练
4. 启动预测服务

#### 训练

下载源码

```shell
git clone git@github.com:xizixuejie/PaddleNLP.git
```

修改为自己的数据集

数据集格式

qid - 序号， label - 0 负面 1 正向

dev.tsv

```
qid	label	text_a
0	1	這間酒店環境和服務態度亦算不錯,但房間空間太小~~不宣容納太大件行李~~且房間格調還可以~~ 中餐廳的廣東點心不太好吃~~要改善之~~~~但算價錢平宜~~可接受~~ 西餐廳格調都很好~~但吃的味道一般且令人等得太耐了~~要改善之~~
1	1	<荐书> 推荐所有喜欢<红楼>的红迷们一定要收藏这本书,要知道当年我听说这本书的时候花很长时间去图书馆找和借都没能如愿,所以这次一看到当当有,马上买了,红迷们也要记得备货哦!
2	0	商品的不足暂时还没发现，京东的订单处理速度实在.......周二就打包完成，周五才发货...
3	1	２００１年来福州就住在这里，这次感觉房间就了点，温泉水还是有的．总的来说很满意．早餐简单了些．
4	1	不错的上网本，外形很漂亮，操作系统应该是个很大的 卖点，电池还可以。
5	0	房间地毯太脏，临近火车站十分吵闹，还好是双层玻璃。服务一般，酒店门口的TAXI讲是酒店的长期合作关系

```

train.tsv

```
label	text_a
1	选择珠江花园的原因就是方便，有电动扶梯直接到达海边，周围餐馆、食廊、商场、超市、摊位一应俱全。
1	15.4寸笔记本的键盘确实爽，基本跟台式机差不多了，蛮喜欢数字小
0	1.接电源没有几分钟,电源适配器热的不行.  
1	今天才知道这书还有第6卷,真有点郁闷:为什么同一套书有两种版本呢?当当网是不是该跟出版社商量商量,单独出个第6卷,让我们的孩子不会有所遗憾。
0	机器背面似乎被撕了张什么标签，残胶还在。但是又看不出是什么标签不见了，该有的都在，怪
```

test.tsv

```
qid	text_a
0	这个宾馆比较陈旧了，特价的房间也很一般。总体来说一般
1	怀着十分激动的心情放映，可是看着看着发现，在放映完毕后，出现一集米老鼠的动画片！
2	还稍微重了点，可能是硬盘大的原故，还要小
4	不错， 
5	有了第一本书的铺垫，读第二本的 
6	前台接待太差，酒店有A B楼之分，本 
7	1. 白色的，很漂亮，做工还可以； 
```



修改 `examples/text_classification/rnn/train.py` 中第90行

```python
# Loads dataset.
train_ds, dev_ds = load_dataset("chnsenticorp", splits=["train", "dev"])
```

修改为

```java
train_ds, dev_ds = load_dataset('chnsenticorp', data_files=['custom-dataset/train.tsv', 'custom-dataset/dev.tsv'], splits=['train', 'dev'])
```

其中 `data_files` 为自定义数据集的位置。需要和 `splits` 参数中的数组对应。

执行以下命令开始训练

```shell
python train.py \
	--vocab_path='./vocab.json' \
	--device=cpu \
	--network=bilstm \
	--lr=5e-4 \
	--batch_size=64 \
	--epochs=10 \
 	--save_dir='./checkpoints'
```

参数说明：

- `vocab_path`: 用于保存根据语料库构建的词汇表的文件路径。
- `device`: 选用什么设备进行训练，可选cpu、gpu或者xpu。如使用gpu训练则参数gpus指定GPU卡号。目前xpu只支持模型网络设置为lstm。
- `network`: 模型网络名称，默认为`bilstm`， 可更换为bilstm，bigru，birnn，bow，lstm，rnn，gru，bilstm_attn，cnn等。
- `lr`: 学习率， 默认为5e-5。
- `batch_size`: 运行一个batch大小，默认为64。
- `epochs`: 训练轮次，默认为10。
- `save_dir`: 训练保存模型的文件路径。
- `init_from_ckpt`: 恢复模型训练的断点路径。

### 导出模型

可选

```shell
python export_model.py \
	--vocab_path=./vocab.json \
	--network=bilstm \
	--params_path=./checkpoints/final.pdparams \
	--output_path=./static_graph_params
```



### 预测服务使用

下载源码

```shell
git clone git@gitlab.laiease.com:hanzhengjie/emotion_server.git
```

执行以下命令启动服务

```shell
cd emotion_server
docker-compose up -d
```

docker-compose.yaml 配置

```yaml
version: '3'

services:
  emotion-server:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: emotion-server
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - ./emotion/pdparams:/opt/emotion/pdparams
```

pdparams 文件夹内的内容为训练的时候生产的 `final.pdparams` 和 `vocab.json`



请求接口 `post /predict`

请求参数：

```json
[
	"很好",
    "不会再买了"
]
```

返回示例：

```json
{
	"data": [
		"positive",
		"negative"
	],
	"success": true
}
```

