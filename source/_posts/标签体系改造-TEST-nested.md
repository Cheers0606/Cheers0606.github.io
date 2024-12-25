---
title: 标签体系改造-TEST-nested
tags: ["trident","label","标签"]
overdue: false
categories: trident
date: 2021-01-19 17:49:48
updated: 2021-01-19 17:49:48
---
## 创建mapping并插入数据 
```bash
PUT test4
{
  "mappings": {
    "label_default": {
      "_all": {
        "enabled": false
      },
      "dynamic_templates": [
        {
          "time": {
            "match": "*_time",
            "match_mapping_type": "string",
            "mapping": {
              "format": "strict_date_optional_time||yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
              "type": "date"
            }
          }
        },
        {
          "timestamp": {
            "match": "*_time",
            "match_mapping_type": "long",
            "mapping": {
              "format": "strict_date_optional_time||epoch_millis",
              "type": "date"
            }
          }
        },
        {
          "default": {
            "match": "*",
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
            }
          }
        }
      ],
      "properties": {
        "tree_1": {
          "type": "nested",
          "properties": {
            "expire_time": {
              "type": "date"
            },
            "id": {
              "type": "long"
            },
            "update_time": {
              "type": "date"
            },
            "value": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        }
      }
    }
  }
}

PUT test4/label_default/34de9d4d3da142348ad8f4e23e05d71c/_create
{
  "tree_1": [
    {
      "id": 1641,
      "value": "客户有下单",
      "update_time": 1610990713000,
      "expire_time": 4102416000000
    }
  ]
}


PUT test4/label_default/28a47dc2f4814e70b95b4f69bd592bbd/_create
{
  "tree_1": [
    {
      "id": 1641,
      "value": "客户有下单",
      "update_time": 1610990713000,
      "expire_time": 4102416000000
    },
    {
      "id": 1953,
      "value": "xdd_标签任务1",
      "update_time": 1610990713000,
      "expire_time": 4102416000000
    }
  ]
}
```

## hive 刷数据到 es
```sql
# es.mapping.parent和es.mapping.routing是针对父子文档的设置
DROP TABLE IF EXISTS db_datastory_trident.hive2es_test4;

CREATE EXTERNAL TABLE db_datastory_trident.hive2es_test4(user_id string,tree array<struct<id:bigint,value:string,update_time:timestamp,expire_time:timestamp>>) 
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler' 
TBLPROPERTIES("es.nodes"="dev1:9203,dev1:9204", "es.write.operation"="upsert", "es.update.retry.on.conflict"="10", "es.resource"="test4/label_default", "es.mapping.id"="user_id", "es.mapping.names"="user_id:user_id,tree:tree_1");

# collect为UDAF，用于将group by外的字段组成一个array。（collect_set增强）
# add jar hdfs:///user/hive/ds-trident-serv-etl-1.2.1.1-SNAPSHOT.jar;
# drop temporary function if exists collect;
# create temporary function collect as 'com.datastory.trident.serv.udf.CollectUDAF';


INSERT INTO TABLE db_datastory_trident.hive2es_test4
SELECT entity_id ,
       collect(named_struct("id", label_id,"value", label_val, "update_time", create_time,"expire_time",expire_time)) AS tree
FROM all_temp_tags where entity_type=497 and label_date='2021-01-21' 
GROUP BY entity_id;


# 考虑all_temp_tags中增加tree的分区，entity_type为1级分区，tree为2级分区，label_date为3级分区，以树为单位刷数据
```
## 更新单个标签
```sql
1953

#INSERT INTO TABLE db_datastory_trident.hbase_label_415_temp_1479 SELECT entity_id,label_val,create_time,expire_time FROM db_datastory_trident.all_temp_tags WHERE entity_type=415 AND label_date='#{#cur_date}' AND label_id=1479;

DROP TABLE IF EXISTS db_datastory_trident.tmp_trident_es_497_label_1953;

CREATE EXTERNAL TABLE db_datastory_trident.tmp_trident_es_497_label_1953(user_id string,id bigint,value string,update_time timestamp,expire_time timestamp) 
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler' 
TBLPROPERTIES(
"es.nodes"="dev1:9203,dev1:9204",
"es.write.operation"="upsert",
"es.update.retry.on.conflict"="10",
"es.resource"="test4/label_default",
"es.mapping.id"="user_id"
"es.mapping.names"="user_id:user_id,id:tree_1.id,value:tree_1.value,update_time:tree_1.update_time,expire_time:tree_1.expire_time"
);

DROP TABLE IF EXISTS db_datastory_trident.tmp_trident_es_497_label_1953;
CREATE EXTERNAL TABLE db_datastory_trident.tmp_trident_es_497_label_1953(user_id string,tree struct<id:bigint,value:string,update_time:timestamp,expire_time:timestamp>) 
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler' 
TBLPROPERTIES("es.nodes"="dev1:9203,dev1:9204", "es.write.operation"="upsert", "es.update.retry.on.conflict"="10", "es.resource"="test4/label_default", "es.mapping.id"="user_id", "es.mapping.names"="user_id:user_id,id:id,value:value,update_time:update_time,expire_time:expire_time");


INSERT INTO TABLE db_datastory_trident.tmp_trident_es_497_label_1953

 select user_id,label from (SELECT user_id,tree from db_datastory_trident.tmp_trident_es_497_label_1953 where user_id in (SELECT entity_id as user_id FROM db_datastory_trident.all_temp_tags WHERE entity_type=497 AND label_date='2021-01-21' AND label_id=1953)) as tmp
 lateral view explode(tree) col as label;
 SELECT entity_id as user_id,1953 as id,label_val as value,create_time as update_time,expire_time FROM db_datastory_trident.all_temp_tags WHERE entity_type=497 AND label_date='2021-01-21' AND label_id=1953;


SELECT pageid, adid 
    FROM pageAds LATERAL VIEW explode(adid_list) adTable AS adid;

select user_id,label from db_datastory_trident.tmp_trident_es_497_label_1953  LATERAL VIEW explode(tree) tree_table as label limit 10;

```
## hive 刷数据到 HBase

## 查询
注意查询需要用nested类型
```bash
GET test4/_search
{
  "query": {
    "nested": {
      "path": "tree_1",
      "query": {
        "term": {
          "tree_1.id": {
            "value": "1953"
          }
        }
      }
    }
  }
}
```
