# xiaoming-test
just for test
#### 1. 这里是一级页面显示的数据内容，主要是total相关的统计

|字段| 描述|
|:------:| :-----:|:----:|
|*一级展示列表*|
|company|公司名字|
|date_start|开始日期|
|date_end|结束日期|
|keywords_num|关键字总数|
|company total events [C]|公司相关所有events|
|company overall presence ratio|A/B|
|company total records[D]|所有公司相关的records总数|
|company total events records ratio[A] |C/D|
|bid request number|bid总请求数|
|exchange request number|exchange对应请求数，格式如{“openx”:1, ”index”:2}|
|global total events[E]|所有关键字扩展后所有events|
|global total records[F]|完整记录中所有记录|
|global total events record ratio[B]|E/F|
|campaign|campaign  ID|123|

```
company total records，company total events records ratio，bid request number，exchange request number，

```

#### 2. 这里是二级页面显示的数据内容，主要是keyword 相关的统计

|字段|描述|
|:------:| :-----:| :----:|
|*二级展示列表*|
|keyword|关键字|
|start_end_date|时间范围|



|mounth_1_num|第一个月数|
|mounth_2_num|第二个月数|
|mounth_n_num|第 n个月数|
|Company keyword events[C]|一个关键字所有与公司相关的events|
|Company  keyword presence ratio|A/B|
|global keyword events[D]|一个关键字扩展后所有events|
|global total records[E]|完整记录中所有记录|
|global  keyword events records  ratio[B]|D/E|
|company total   records[F]|所有与公司相关的records|
|company keyword events record ratio[A]|C/F|

  |字段| 描述|例如|
  |:--------:| :-----:| :----: |
  | *company_records *|
  |day|日期|2017-05-02|
  |company|公司名|IBM|
  |records|请求数|1234|
