---
layout:       post
title:        "ElastAlert配置(译)"
subtitle:     "ElastAlert"
date:         2018-06-26 16:00:00
author:       "coderm520"
header-img:   ""
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - ELK
---

- 创建index: $ elastalert-create-index

- 创建一个rule

>Each rule defines a query to perform, parameters on what triggers a match, and a list of alerts to fire for each match. We are going to use example_rules/example_frequency.yaml as a template:

- 每个规则定义一个要执行的查询，触发匹配的参数以及每个匹配触发的警报列表。我们将使用example_rules/example_frequency.yaml作为模板:

- From example_rules/example_frequency.yaml
```yaml
es_host: elasticsearch.example.com
es_port: 14900
name: Example rule
type: frequency
index: logstash-*
num_events: 50
timeframe:
    hours: 4
filter:
 term:
    some_field: "some_value"
alert:
 "email"
email:
 "elastalert@example.com"
```

## config.yaml
>rules_folder is where ElastAlert will load rule configuration files from. It will attempt to load every .yaml file in the folder. Without any valid rules, ElastAlert will not start. ElastAlert will also load new rules, stop running missing rules, and restart modified rules as the files in this folder change. For this tutorial, we will use the example_rules folder

- rules_folder：ElastAlert将从此文件夹加载规则配置文件，它试图在此文件夹中加载所有的yaml后缀文件。如果没有可用的规则，ElastAlert不会启动。如果文件夹中的文件发生变化，ElastAlert也会相应做出反应（如加载新规则、停止运行丢失的规则、重启有修改的规则等）。本教程中，我们使用example_rules文件夹

>run_every is how often ElastAlert will query Elasticsearch

- run_every: ElastAlert查询Elasticsearch的频率

>buffer_time is the size of the query window, stretching backwards from the time each query is run. This value is ignored for rules where use_count_query or use_terms_query is set to true

- buffer_time是查询窗口的时间间隔，从每个查询运行的时间向前延伸。对于use_count_query或use_terms_query设置为true的规则，此值将被忽略


http://elastalert.readthedocs.io/en/latest/running_elastalert.html#setting-up-elasticsearch   待添加

##　配置详细
>es_host and es_port should point to the Elasticsearch cluster we want to query
- es_host和es_port应该指向我们要查询的Elasticsearch集群

>name is the unique name for this rule. ElastAlert will not start if two rules share the same name
- name是此规则的唯一名称。如果两个规则共享相同的名称，ElastAlert将不会启动

>type: Each rule has a different type which may take different parameters. The frequency type means “Alert when more than num_events occur within timeframe.” For information other types, see Rule types.
- type:每个规则都有一个不同的类型，可能会采用不同的参数。'frequency'类型表示“在时间范围内发生超过num_events数目事件时发出警报”。有关其他类型的信息，请参阅[规则类型](http://elastalert.readthedocs.io/en/latest/ruletypes.html#ruletypes "规则类型")

>index: The name of the index(es) to query. If you are using Logstash, by default the indexes will match "logstash-*".
- index:要查询的索引名称。如果您正在使用Logstash，默认情况下索引将匹配“logstash- *”

>num_events: This parameter is specific to frequency type and is the threshold for when an alert is triggered
- num_events：该参数特定于频率类型，并且是触发警报时的阈值

>timeframe is the time period in which num_events must occur
- timeframe:时间范围,是num_events必须发生的时间段

>filter is a list of Elasticsearch filters that are used to filter results. Here we have a single term filter for documents with some_field matching some_value. See Writing Filters For Rules for more information. If no filters are desired, it should be specified as an empty list: filter: []

- filter: Elasticsearch过滤器的列表,用于过滤结果.我们为some_field与some_value匹配的文档设定了单项过滤器.请参阅[编写规则过滤器](http://elastalert.readthedocs.io/en/latest/recipes/writing_filters.html#writingfilters "编写规则过滤器")以查看更多信息

>alert is a list of alerts to run on each match. For more information on alert types, see Alerts. The email alert requires an SMTP server for sending mail. By default, it will attempt to use localhost. This can be changed with the smtp_host option

- alert是每个匹配运行的警报列表。有关警报类型的更多信息，请参阅[Alerts](http://elastalert.readthedocs.io/en/latest/ruletypes.html#alerts "Alerts")。电子邮件警报需要SMTP服务器来发送邮件。默认情况下，它会尝试使用本地主机。这可以通过smtp_host选项进行更改

>email is a list of addresses to which alerts will be sent
email是警报将发送到的地址列表

- 还有许多其他可选配置选项，请参阅常见[配置选项](http://elastalert.readthedocs.io/en/latest/ruletypes.html#commonconfig "配置选项")

>All documents must have a timestamp field. ElastAlert will try to use @timestamp by default, but this can be changed with the timestamp_field option. By default, ElastAlert uses ISO8601 timestamps, though unix timestamps are supported by setting timestamp_type
As is, this rule means “Send an email to elastalert@example.com when there are more than 50 documents with some_field == some_value within a 4 hour period.”

- 所有配置文档都必须有timestamp(时间戳)字段。 ElastAlert默认会尝试使用@timestamp,
但是这可以通过timestamp_field选项进行更改. 默认情况下，ElastAlert使用ISO8601时间戳，但通过设置timestamp_type支持unix时间戳
这个规则的意思是“如果在4小时内有超过50个包含some_field == some_value的文档，发送电子邮件到elastalert@example.com。”

 timestamp_field: "@timestamp"
 timestamp_type: unix_ms 

ps:timestamp_field入库字段需匹配timestamp_type时间格式

## 测试你的Rule

>Running the elastalert-test-rule tool will test that your config file successfully loads and run it in debug mode over the last 24 hours

- 运行elastalert-test-rule工具将测试您的配置文件是否成功加载并在过去24小时内以调试模式运行

`$ elastalert-test-rule example_rules/example_frequency.yaml`

>If you want to specify a configuration file to use, you can run it with the config flag:

- 如果要指定要使用的配置文件，可以使用config标志运行它
`$ elastalert-test-rule --config <path-to-config-file> example_rules/example_frequency.yaml`

>The configuration preferences will be loaded as follows:
1. Configurations specified in the yaml file.
2. Configurations specified in the config file, if specified.
3. Default configurations, for the tool to run.

- 配置首选项将如下加载：
1. yaml文件中指定的配置。 
2. 如果指定，则在配置文件中指定配置。 
3. 默认配置，供工具运行

- 更多信息请查看 the testing section for more details[testing](http://elastalert.readthedocs.io/en/latest/ruletypes.html#testing "testing")


## 运行ElastAlert

>There are two ways of invoking ElastAlert. As a daemon, through Supervisor (http://supervisord.org/), or directly with Python. For easier debugging purposes in this tutorial, we will invoke it directly

- 有两种调用ElastAlert的方法。作为守护进程，可以通过Supervisor或直接使用Python。为了便于在本教程中进行调试，我们将直接调用它

`
$ python -m elastalert.elastalert --verbose --rule example_frequency.yaml  # or use the entry point: elastalert --verbose --rule ...
No handlers could be found for logger "Elasticsearch"
INFO:root:Queried rule Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 hits
INFO:Elasticsearch:POST http://elasticsearch.example.com:14900/elastalert_status/elastalert_status?op_type=create [status:201 request:0.025s]
INFO:root:Ran Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 query hits (0 already seen), 0 matches, 0 alerts sent
INFO:root:Sleeping for 297 seconds
`

>ElastAlert uses the python logging system and --verbose sets it to display INFO level messages. --rule example_frequency.yaml specifies the rule to run, otherwise ElastAlert will attempt to load the other rules in the example_rules folder.

- ElastAlert使用python日志记录系统，--verbose将其设置为显示INFO级别的消息。 --rule example_frequency.yaml指定要运行的规则，否则ElastAlert将尝试加载example_rules文件夹中的其他规则。

>Let’s break down the response to see what’s happening.
`Queried rule Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 hits`

ElastAlert periodically queries the most recent buffer_time (default 45 minutes) for data matching the filters. Here we see that it matched 5 hits.

- 让我们分解response，看看发生了什么

ElastAlert定期查询最近的buffer_time（默认为45分钟），以查找与过滤器匹配的数据。在这里我们看到它匹配了5次



`POST http://elasticsearch.example.com:14900/elastalert_status/elastalert_status?op_type=create [status:201 request:0.025s]`

> This line showing that ElastAlert uploaded a document to the elastalert_status index with information about the query it just made.
- 此行显示ElastAlert将文档上传到了elastalert_status索引，并提供了有关它刚刚创建的查询的信息



`Ran Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 query hits (0 already seen), 0 matches, 0 alerts sent`

>The line means ElastAlert has finished processing the rule. For large time periods, sometimes multiple queries may be run, but their data will be processed together. 
query hits is the number of documents that are downloaded from Elasticsearch, 
already seen refers to documents that were already counted in a previous overlapping query and will be ignored, matches is the number of matches the rule type outputted, 
and alerts sent is the number of alerts actually sent. 
This may differ from matches because of options like realert and aggregation or because of an error

- 该行意味着ElastAlert已完成处理规则。对于大量的时间段，有时可能会运行多个查询，但他们的数据将一起处理.
query hits是从Elasticsearch下载的文档数量，
already seen是指已经在上一个重叠查询中计数并且将被忽略,matches是输出规则类型的匹配数量，
alerts sent是实际发送的警报的数量
这可能会因为realert和aggregation 等选项或者错误而不同

`Sleeping for 297 seconds`

>The default run_every is 5 minutes, meaning ElastAlert will sleep until 5 minutes have elapsed from the last cycle before running queries for each rule again with time ranges shifted forward 5 minutes.

- 默认的run_every是5分钟，这意味着ElastAlert将休眠直到从上一个周期开始经过5分钟，然后再次运行每个规则的查询，同时时间范围向前移动5分钟

>Say, over the next 297 seconds, 46 more matching documents were added to Elasticsearch:
- 假设在接下来的297秒内，还有46个匹配的文件被添加到Elasticsearch中

`INFO:root:Queried rule Example rule from 1-15 14:27 PST to 1-15 15:12 PST: 51 hits
...
INFO:root:Sent email to ['elastalert@example.com']
...
INFO:root:Ran Example rule from 1-15 14:27 PST to 1-15 15:12 PST: 51 query hits, 1 matches, 1 alerts sent`


>The body of the email will contain something like:
- 电子邮件的正文将包含如下内容：

`Example rule
At least 50 events occurred between 1-15 11:12 PST and 1-15 15:12 PST
@timestamp: 2015-01-15T15:12:00-08:00`

>If an error occurred, such as an unreachable SMTP server, you may see:
- 如果发生错误，例如无法访问的SMTP服务器，您可能会看到：
`ERROR:root:Error while running alert email: Error connecting to SMTP host: [Errno 61] Connection refused`


>Note that if you stop ElastAlert and then run it again later, it will look up elastalert_status and begin querying at the end time of the last query. This is to prevent duplication or skipping of alerts if ElastAlert is restarted.

By using the --debug flag instead of --verbose, the body of email will instead be logged and the email will not be sent. In addition, the queries will not be saved to elastalert_status.

- 请注意，如果您停止ElastAlert并稍后再运行它，它将查找elastalert_status并在最后一次查询的结束时开始查询。这是为了防止在ElastAlert重新启动时重复或跳过警报
通过使用--debug标志而不是--verbose，电子邮件的主体将被记录，并且电子邮件将不会被发送。另外，查询不会保存到elastalert_status

- 邮件配置 
`
    es_host: 192.168.137.101
    es_port: 9200
    name: my frequency rule
    type: frequency
    index: account*
    num_events: 3
    timeframe:
        minutes: 10

    timestamp_type: unix_ms
    timestamp_field: "@timestamp"

    filter:
        - query_string:
            query: "number: >10"


    alert:
        - "email"

    email:
        - "xxxx@xingyuanauto.com"

    smtp_ssl: true
    smtp_host: smtp.163.com
    smtp_port: 465
    alert_text: "test elastalert to you"
    smtp_auth_file: smtp_auth_file.yaml
    from_addr: "miaoxxxx@163.com"
`

- smtp_auth_file.yaml
    user：xxxx
    password: xxxx

- 例：buffer_time:days 1, run_every:seconds:20,timeframe:minutes 10, num_events:3
    每20秒（run_every）查询一次es，只查询从现在开始往后30分钟（例：buffer_time）的数据，10分钟内（timeframe：从当前查询执行时间点往前10分钟内）如果有满足条件数目（num_events）的查询（条件为filter），触发警报

- ps：unix时间戳转换http://tool.chinaz.com/Tools/unixtime.aspx