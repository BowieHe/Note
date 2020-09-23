## saveWechatmassSend:

save the weChat mass info
wmsCampaignMap -> id, wmsCampaignInfo -> campaignid,uuid,groupid
wmsList -> 所有文章的信息

更新图文消息状态，更新为ready，create_group（有abTest）





sendByOpenIdList中，如果有abtest，每个信息里的abGroups会不会为空



启动一个sparkJob，根据传递给sparkJob的JSON参数不同，启动不同的sparkJob

新建两个sparkJob，一个sparkJob用于获取选定的customerID和openID

另一个sparkJob用于存储全量粉丝信息，并且



```groovy
req = [
pageSize : pageSize,
dataTopic: topic,
taskType : "customerExtractor",
transData: [wmsId: wms.id, batchUuid: batchUuid, valuePrefix: valuePrefix, type: "processEventFilter", wmsList: wmsList, isAbTest: wms.isAbTest],
taskDef  : [
segment       : [type:"customer", filter/listId],
constrains    : [wechatAccount: wms.wechatAccount, wechatFansOnly: true],
fields        : [wechatfans: ["customerId", "openid"]]
]
// 添加
taskId				:uuid,
statusUpdateTopic: "",
type					:"start",

];

fillSegment only process the data with abtest
```

```
def messageData = [
job_name    : spark-customer-extractor,
cdh_version : cdhVersion,
hadoop_node : hadoopNode,
deploy_env  : System.getProperty("grails.env"),
jar_name    : spark-customer-extractor,
wait_timeout: waitTimeout
]
```

```groovy
["listId"  : abTestMassSend.sendGroupList,
"taskType": "split_percentage",
"taskDef" : [
"lists"    : [name: it.name, value: (it.rate / 100).toString(), massSendId: it.id],
"topic"    : WECHAT_FANS_FILTER,
"transData": ["batchUuid": batchUuid,
  "tenantId" : currentTenant.get(),
  "sendType" : "byOpenId"]
	]
]

spark = [
  jobName: spark-list-operation,
  jarName:spark-list-operation,
  data = data,
  replica: 3,
  type: 'start'
]
```

```
select open_id, customer_id from wechat_user_tag where batch_uuid = :batchUuid and wechat_mas_send_id = :wechatMassSendId and status = :status and open_id > :openId order by open_id limit ${sendLimit}"
```

message.isStatic

通过lb的过程获取到customerId and openId

将对应的人存入parquet表