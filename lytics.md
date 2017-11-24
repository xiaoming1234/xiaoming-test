# social media 后端处理设计文档

-  结构简单描述 
-  数据库设计
-  s3 目录规划
-  job调度
-  task结果处理


## 1. 结构简单描述
   - 前端负责资源录入：company、title,状态显示；
   - 后端负责实际业务的调度，管理，数据处理等；
   当前后端任务调度采用crontab job 扫描 sme_task 数据来完成，任务分为两种：
    1. spider 任务，串行进行，（250条）一个任务
    2. 结果处理任务 尽量并行进行。


## 2. 数据库设计
- sme_task

| 字段        | 类型           | 描述  |
| ------------- |:-------------:| -----:|
| campaign_id     | int | campaign id |
| task_id     | int      |  task id |
| type | enum('gsearch_personal','gsearch_domain','gsearch_url','lsearch_url','lsearch_company')     |    任务类型|
|task_input|varchar(256)|任务输入数据|
|spider_result|varchar(256)|spider 结果|
|task_result|varchar(256)|处理后结果|
|status|enum('spider_ready','spider_running','spider_finish','process_running','finish')|任务状态|

- sme_campaign_title
 
| 字段        | 类型           | 描述  |
| ------------- |:-------------:| -----:|
| campaign_id     | int | campaign id |
| title     | vchar     |  title name |
| ranke     | int      |  title ranke |


- sme_campaign
 
| 字段        | 类型           | 描述  |
| ------------- |:-------------:| -----:|
| campaign_id     | int | campaign id |
| campaign_name     | vchar     |  campaign name |
| create_time     | datetime     |  创建时间 |


- sme_campaign_list
 
| 字段        | 类型           | 描述  |
| ------------- |:-------------:| -----:|
| campaign_id     | int | campaign id |
| list_id     | int     | list id |


- sme_list_member
 
| 字段        | 类型           | 描述  |
| ------------- |:-------------:| -----:|
| _list_id     | int | list id |
| value     | vchar     | list 值 |
 

说明：

1. sme_task表管理任务状态及结果
2. sme_campaign_title campaign表是title资源表
3. sme_campaign表是campaign信息
4. sme_campaign_list、sme_list_member 公司资源的管理


## 3. s3目录规划
### 3.1 task finished processing result path:
     s3://bilin-prod/social_media/data/campaign_id/task_id/type_date.data

### 3.2 task job 数据
    s3://bilin-prod/social_media/data/campaign_id/tasks/lytics_gsearch_domain_1

### 3.3 campaign 公共数据（title\_list,company\_list）
    s3://bilin-prod/social_media/data/campaign_id/company_list
    s3://bilin-prod/social_media/data/campaign_id/title_list

### 3.4 所有campaign 公共数据
    title 简称-全称 映射表：
    s3://bilin-prod/social_media/data/common/title_short_list

## 4. job 调度
   分两部分：spider 任务调度 和 处理流程调度
### 4.1 spider 任务调度
    1. gsearch user:
        该任务没有依赖，任务数目较大：total= company数 * title数据，任务串行执行，一个job250个任务，调度时按照job id 顺序，依次进行。
    2. gsearch domain:
        该任务没有依赖，任务数目较大：total= company数 ，任务串行执行，一个job250个任务，调度时按照job id 顺序，依次进行。
    3. gsearch linkedin company url
        该任务依赖2中domain,任务数最大为：total= company数，数任务串行执行，一个job250个任务，调度时按照job id 顺序依次进行。
    4. linkedin search company url
        该任务没有依赖，但是结果提取不理想，一般做备份处理，任务数最大为：total= company数，数任务串行执行，一个job250个任务，
        调度时按照job id 顺序，依次进行。
    5. linkedin likes company infomation
        该任务依赖3和4中得到的url,任务数最大为：total= company数，数任务串行执行，一个job250个任务，调度时按照job id 顺序，依次进行。
    
### 4.2 流程调度
    1. gsearch user:
         该任务处理流程依赖2,3,4,5的执行，任务横向无依赖，如果2,3,4,5已经完成，该任务完成一个处理一个。
    2. gsearch domain:
         该任务处理流程横向纵向均无依赖，任务完成一个处理一个；
    3. gsearch linkedin company url
         该任务处理流程横向纵向均无依赖，任务完成一个处理一个；
    4. linkedin search company url
         该任务处理流程横向纵向均无依赖，任务完成一个处理一个；
    5. linkedin likes company infomation
         该任务处理流程横向纵向均无依赖，任务完成一个处理一个；

## 5. task 结果处理
### 5.1 gsearch user
    输入：
        task spier result 
        company infomation 数据
        title list
        company list      
        
    输出：(s3 path and mysql)
        s3://bilin-prod/social_media/data/campaign_id/task_id/SME_lytics_gsearch_linkedin_user_4_1510217357973_result
    

    处理流程： 
    -  遍历结果，过滤得到 user, title, company 等信息；
    -  使用campaign title list 打分过滤数据；
       找到campaign 提供的title list 中，最佳匹配的title（单词集合交集为提供title中的项，找不到则其分数为0）, 然后计算交集单词数/并集单词数，得比例分数
    -  使用campaign campany list 打分过滤数据；
    -  添加公司相关信息，email信息等；
    -  结果写入 s3 和 MySQL；
### 5.2 gsearch domain

    输入：
        task spier result
    输出：
        s3

    处理流程：
	- 格式化task结果
	- 使用模型，得到domain
	- 结果（company，domain）写入s3结果目录

### 5.3 gsearch linkedin company url
    输入：
        task spier result
    输出：
        s3 result 目录
    
    处理流程：
    - 遍历结果数据，分析url
    - 找到的第一个含有"https://www.linkedin.com/company/"的url作为当前搜索得到的有效link
    - 将（domain,url) 写入s3 结果目录


### 5.4 linkedin search company url
     输入：
        task spier result 
     输出：
        s3 result 目录

     处理流程：
     - 遍历结果，分析keyword 与company name
     - 如果结果只有一条记录，则该链接就是我们需要的
     - 如果结果有多条记录，只有一条名字与关键字相同，则该记录是我们需要的
     - 其它情况放弃
     - 结果（company,url)写入s3 结果目录
     - 
### 5.5 linkedin likes company infomation 
    输入：
       task spier result    
    输出：
       s3 结果目录

    处理流程
    - 遍历结果数据
    - 结果（url,campaign infotion) 写入s3;
    - 

