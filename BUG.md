#### 程序在V1环境中指向的请求地址已经是V1环境中的地址前缀，但是最后代码请求还是发送到了test环境中

在service_rejection表中添加一条记录，将该地址转发指向V1环境，如果不添加该条记录，会默认指向test环境30000，（V1是30002）





```
cl_bid'        : 'batchId',
'cl_cid'        : 'cl_cid',
'cl_contentName': 'contentName',
'cl_event'      : 'event',
'cl_pageId'     : 'pageId',
'cl_targetId'   : 'targetId',
'cl_targetName' : 'targetName',
'cl_srt'        : 'srt',
'cl_tag'        : 'tag',
'cl_source'     : 'source',
'cl_campaign'   : 'campaign',
'cl_latitude'   : 'latitude',
'cl_longitude'  : 'longitude',
'cl_ipCountry'  : 'ipCountry',
'cl_ipProvince' : 'ipProvince',
'cl_ipCity'     : 'ipCity',
'cl_ipCounty'   : 'ipCounty',
'cl_valueTag'   : 'valueTag'
```



```
"total"    : Integer.parseInt(connection.sync().hget(syncTradeKey, "Total")),
"finished" : Integer.parseInt(connection.sync().hget(syncTradeKey, "Finished")),
"failed"   : Integer.parseInt(connection.sync().hget(syncTradeKey, "Failed")),
"status"   : res.status,
"kdtId"    : res.kdtId,
"shopId"   : res.shopId,
"shopName" : res.shopName
```

设定同步时间和模式

```
def products = []
ordersMap.orders.each {
    products << [
            lineId       : it.oid,
            productName  : it.title,
            productId    : it.item_id,
            skuId        : it.sku_id,
            qty          : it.num,
            priceUnit    : it.price,
            priceSubTotal: it.total_fee,
            priceSubPaid : it.payment
    ]
}

// 提取订单信息
def order = [
        orderNo         : book_key,
        salesChannel    : "youzan",
        contactName     : buyerInfoMap.address.receiver_name,
        contactTel      : buyerInfoMap.buyer.buyer_phone,
        shippingProvince: buyerInfoMap.address.delivery_province,
        shippingCity    : buyerInfoMap.address.delivery_city,
        shippingCounty  : buyerInfoMap.address.delivery_district,
        shippingAddress : buyerInfoMap.address.delivery_address,
        lines           : products
```