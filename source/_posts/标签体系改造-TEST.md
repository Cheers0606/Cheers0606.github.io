---
title: 标签体系改造-TEST
tags: ["trident","label","标签"]
overdue: true
categories: trident
date: 2021-01-19 17:49:48
updated: 2021-01-19 17:49:48
---
# 测试
## 创建es mapping
```bash
PUT /trident_test_497
{
  "settings": {
    "analysis": {
      "analyzer": {
        "vertical": {
          "type": "pattern",
          "parttern": "\\|"
        }
      }
    }
  },
  "mappings": {
    "label_default": {
      "_all": {
        "enabled": false
      },
      "_parent": {
        "type": "info"
      },
      "_routing": {
        "required": true
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
      ]
    },
    "label_basic": {
      "_all": {
        "enabled": false
      },
      "_parent": {
        "type": "info"
      },
      "_routing": {
        "required": true
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
      ]
    },
    "info": {
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
        "activate_days": {
          "type": "integer"
        },
        "age": {
          "type": "integer"
        },
        "birthday": {
          "type": "date"
        },
        "children_size": {
          "type": "integer"
        },
        "city": {
          "type": "keyword"
        },
        "city_level": {
          "type": "keyword"
        },
        "community": {
          "type": "keyword"
        },
        "constellation": {
          "type": "keyword"
        },
        "contact_address": {
          "type": "keyword"
        },
        "contact_email": {
          "type": "keyword"
        },
        "contact_phone": {
          "type": "keyword"
        },
        "contact_telephone": {
          "type": "keyword"
        },
        "created_time": {
          "type": "date"
        },
        "district": {
          "type": "keyword"
        },
        "downgrade_cnt": {
          "type": "integer"
        },
        "education": {
          "type": "keyword"
        },
        "family_annual_income": {
          "type": "integer"
        },
        "family_size": {
          "type": "integer"
        },
        "gender": {
          "type": "integer"
        },
        "group_id": {
          "type": "keyword"
        },
        "industry": {
          "type": "keyword"
        },
        "internet_habits": {
          "type": "keyword"
        },
        "is_activate": {
          "type": "integer"
        },
        "is_downgrade": {
          "type": "integer"
        },
        "is_employee": {
          "type": "integer"
        },
        "is_first_complete_personal_data": {
          "type": "keyword"
        },
        "is_have_children": {
          "type": "integer"
        },
        "is_have_pet": {
          "type": "integer"
        },
        "is_member": {
          "type": "integer"
        },
        "kol_points": {
          "type": "integer"
        },
        "life_cycle": {
          "type": "keyword"
        },
        "loyalty": {
          "type": "integer"
        },
        "marital_status": {
          "type": "keyword"
        },
        "market_area": {
          "type": "keyword"
        },
        "member_duration": {
          "type": "integer"
        },
        "member_level": {
          "type": "keyword"
        },
        "member_source": {
          "type": "keyword"
        },
        "member_status": {
          "type": "keyword"
        },
        "open_ids": {
          "type": "keyword"
        },
        "p_date": {
          "type": "keyword"
        },
        "personal_annual_income": {
          "type": "integer"
        },
        "points_balance": {
          "type": "integer"
        },
        "points_to_expired": {
          "type": "integer"
        },
        "prev_level_change_time": {
          "type": "date"
        },
        "previous_life_cycle": {
          "type": "keyword"
        },
        "previous_member_level": {
          "type": "keyword"
        },
        "profession": {
          "type": "keyword"
        },
        "province": {
          "type": "keyword"
        },
        "receive_address": {
          "type": "keyword"
        },
        "resident_address": {
          "type": "keyword"
        },
        "rfm": {
          "type": "keyword"
        },
        "tb_phone": {
          "type": "keyword"
        },
        "tenant_code": {
          "type": "keyword"
        },
        "update_time": {
          "type": "date"
        },
        "user_id": {
          "type": "keyword"
        },
        "user_name": {
          "type": "keyword"
        },
        "user_source": {
          "type": "keyword"
        },
        "work_address": {
          "type": "keyword"
        },
        "wx_phone": {
          "type": "keyword"
        },
        "wx_union_id": {
          "type": "keyword"
        }
      }
    },
    "doc": {
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
        "id": {
          "type": "keyword"
        },
        "label_1970": {
          "type": "keyword"
        },
        "label_1970_expire_time": {
          "type": "date"
        },
        "label_1970_update_time": {
          "type": "date"
        },
        "parent_id": {
          "type": "keyword"
        }
      }
    },
    "_default_": {
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
      ]
    }
  }
}
```
## 添加一个标签树
```bash
PUT /trident_test_497/label_default/_mapping

{
  "properties": {
    "tree_1_label": {
      "type": "string"
    },
    "tree_1_label_expire_time": {
      "type": "string"
    },
    "tree_1_label_update_time": {
      "type": "string"
    }
  }
}
```
## 刷数据进es
```sql
CREATE EXTERNAL TABLE db_datastory_trident.tmp_trident_es_543_1_label_3275(user_id string,tree_1_label string,tree_1_label_update_time string,tree_1_label_expire_time string) 
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler' 
TBLPROPERTIES(
"es.nodes"="dev1:9203,dev1:9204",
"es.write.operation"="upsert",
"es.update.retry.on.conflict"="10",
"es.resource"="trident_test_497/label_default",
"es.mapping.id"="user_id",
"es.mapping.parent"="user_id",
"es.mapping.routing"="user_id",
"es.mapping.names"="user_id:user_id,tree_1_label:tree_1_label,tree_1_label_update_time:tree_1_label_update_time,tree_1_label_expire_time:tree_1_label_expire_time"
);


INSERT INTO TABLE db_datastory_trident.tmp_trident_es_543_1_label_3275
SELECT entity_id,
       CONCAT_WS('|' ,collect_set(concat(label_id,"_",label_val))),
       CONCAT_WS('|' ,collect_set(concat(label_id,"_",create_time))),
       CONCAT_WS('|' ,collect_set(concat(label_id,"_",expire_time)))
FROM db_datastory_trident.all_temp_tags
WHERE entity_type=497
AND label_date='2021-01-18'
GROUP BY entity_id;
```
## 查询es
```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 10069,
    "max_score": 1,
    "hits": [
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "001522388f3948278e47e047f1b68cf4",
        "_score": 1,
        "_routing": "001522388f3948278e47e047f1b68cf4",
        "_parent": "001522388f3948278e47e047f1b68cf4",
        "_source": {
          "user_id": "001522388f3948278e47e047f1b68cf4",
          "tree_1_label": "2083_xjcTest12",
          "tree_1_label_update_time": "2083_2021-01-18 00:10:05",
          "tree_1_label_expire_time": "2083_2021-01-31 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "00449a157c564f7b957b65fac16bfafd",
        "_score": 1,
        "_routing": "00449a157c564f7b957b65fac16bfafd",
        "_parent": "00449a157c564f7b957b65fac16bfafd",
        "_source": {
          "user_id": "00449a157c564f7b957b65fac16bfafd",
          "tree_1_label": "1957_DDD",
          "tree_1_label_update_time": "1957_2021-01-18 00:01:20",
          "tree_1_label_expire_time": "1957_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "0080cfcd11b649e0b943eba6d67ecde3",
        "_score": 1,
        "_routing": "0080cfcd11b649e0b943eba6d67ecde3",
        "_parent": "0080cfcd11b649e0b943eba6d67ecde3",
        "_source": {
          "user_id": "0080cfcd11b649e0b943eba6d67ecde3",
          "tree_1_label": "1957_DDD",
          "tree_1_label_update_time": "1957_2021-01-18 00:01:20",
          "tree_1_label_expire_time": "1957_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "008512c62be741699d16a13e809c9223",
        "_score": 1,
        "_routing": "008512c62be741699d16a13e809c9223",
        "_parent": "008512c62be741699d16a13e809c9223",
        "_source": {
          "user_id": "008512c62be741699d16a13e809c9223",
          "tree_1_label": "1957_DDD",
          "tree_1_label_update_time": "1957_2021-01-18 00:01:20",
          "tree_1_label_expire_time": "1957_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "008f2445b31e463a8a15f4c2a92e86f4",
        "_score": 1,
        "_routing": "008f2445b31e463a8a15f4c2a92e86f4",
        "_parent": "008f2445b31e463a8a15f4c2a92e86f4",
        "_source": {
          "user_id": "008f2445b31e463a8a15f4c2a92e86f4",
          "tree_1_label": "1622_男|1857_男|1984_男|2093_男|2193_男|2306_男",
          "tree_1_label_update_time": "1622_2021-01-18 00:22:46|1857_2021-01-18 00:13:23|1984_2021-01-18 00:16:03|2093_2021-01-18 01:01:43|2193_2021-01-18 00:54:38|2306_2021-01-18 01:42:46",
          "tree_1_label_expire_time": "1622_2100-01-01 00:00:00|1857_2100-01-01 00:00:00|1984_2100-01-01 00:00:00|2093_2100-01-01 00:00:00|2193_2100-01-01 00:00:00|2306_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "00e5f18ba0444aa3968b6c226a59212d",
        "_score": 1,
        "_routing": "00e5f18ba0444aa3968b6c226a59212d",
        "_parent": "00e5f18ba0444aa3968b6c226a59212d",
        "_source": {
          "user_id": "00e5f18ba0444aa3968b6c226a59212d",
          "tree_1_label": "1954_DD",
          "tree_1_label_update_time": "1954_2021-01-18 00:05:44",
          "tree_1_label_expire_time": "1954_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "00ea8cbe08a64b74b205f20f13047fb5",
        "_score": 1,
        "_routing": "00ea8cbe08a64b74b205f20f13047fb5",
        "_parent": "00ea8cbe08a64b74b205f20f13047fb5",
        "_source": {
          "user_id": "00ea8cbe08a64b74b205f20f13047fb5",
          "tree_1_label": "1957_DDD",
          "tree_1_label_update_time": "1957_2021-01-18 00:01:20",
          "tree_1_label_expire_time": "1957_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "010ccd76f83e4d4ea90714eab2b82b81",
        "_score": 1,
        "_routing": "010ccd76f83e4d4ea90714eab2b82b81",
        "_parent": "010ccd76f83e4d4ea90714eab2b82b81",
        "_source": {
          "user_id": "010ccd76f83e4d4ea90714eab2b82b81",
          "tree_1_label": "1953_xdd_标签任务1|1957_DDD",
          "tree_1_label_update_time": "1953_2021-01-18 00:05:37|1957_2021-01-18 00:01:20",
          "tree_1_label_expire_time": "1953_2100-01-01 00:00:00|1957_2100-01-01 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "010ce418ee5c4f22b92482335885f2ff",
        "_score": 1,
        "_routing": "010ce418ee5c4f22b92482335885f2ff",
        "_parent": "010ce418ee5c4f22b92482335885f2ff",
        "_source": {
          "user_id": "010ce418ee5c4f22b92482335885f2ff",
          "tree_1_label": "1953_xdd_标签任务1|2083_xjcTest12",
          "tree_1_label_update_time": "1953_2021-01-18 00:05:37|2083_2021-01-18 00:10:05",
          "tree_1_label_expire_time": "1953_2100-01-01 00:00:00|2083_2021-01-31 00:00:00"
        }
      },
      {
        "_index": "trident_test_497",
        "_type": "label_default",
        "_id": "011a3638369542f39a2b40a012a1212a",
        "_score": 1,
        "_routing": "011a3638369542f39a2b40a012a1212a",
        "_parent": "011a3638369542f39a2b40a012a1212a",
        "_source": {
          "user_id": "011a3638369542f39a2b40a012a1212a",
          "tree_1_label": "1641_客户有下单|1780_客户有下单|1876_客户有下单|1953_xdd_标签任务1|1971_wce|2003_客户有下单|2112_客户有下单|2212_客户有下单|2325_客户有下单",
          "tree_1_label_update_time": "1641_2021-01-18 00:48:54|1780_2021-01-18 00:14:47|1876_2021-01-18 00:38:08|1953_2021-01-18 00:05:40|1971_2021-01-18 00:02:37|2003_2021-01-18 00:40:17|2112_2021-01-18 01:26:53|2212_2021-01-18 01:21:08|2325_2021-01-18 02:05:59",
          "tree_1_label_expire_time": "1641_2100-01-01 00:00:00|1780_2100-01-01 00:00:00|1876_2100-01-01 00:00:00|1953_2100-01-01 00:00:00|1971_2100-01-01 00:00:00|2003_2100-01-01 00:00:00|2112_2100-01-01 00:00:00|2212_2100-01-01 00:00:00|2325_2100-01-01 00:00:00"
        }
      }
    ]
  }
}
```

## 统计
### 标签覆盖人数
```bash
GET trident_test_497/label_default/_search
{
  "query": {
    "match": {
      "tree_1_label": "1780_客户有下单"
    }
  }
}
```
### 树覆盖人数
```bash
GET trident_test_497/label_default/_search
{
  "query": {
    "exists":{
      "field":"tree_1_label"
    }
  }
}
```
### 总人数
```bash
GET trident_test_497/info/_search

```
## 刷数据进es（单个标签）
```sql
CREATE EXTERNAL TABLE db_datastory_trident.tmp_trident_es_543_1_label_3275(user_id string,tree_1_label string,tree_1_label_update_time string,tree_1_label_expire_time string) 
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler' 
TBLPROPERTIES(
"es.nodes"="dev1:9203,dev1:9204",
"es.write.operation"="upsert",
"es.update.retry.on.conflict"="10",
"es.resource"="trident_test_497/label_default",
"es.mapping.id"="user_id",
"es.mapping.parent"="user_id",
"es.mapping.routing"="user_id",
"es.mapping.names"="user_id:user_id,tree_1_label:tree_1_label,tree_1_label_update_time:tree_1_label_update_time,tree_1_label_expire_time:tree_1_label_expire_time"
);


INSERT INTO TABLE db_datastory_trident.tmp_trident_es_543_1_label_3275
SELECT entity_id,
       CONCAT_WS('|' ,collect_set(concat(label_id,"_",label_val))),
       CONCAT_WS('|' ,collect_set(concat(label_id,"_",create_time))),
       CONCAT_WS('|' ,collect_set(concat(label_id,"_",expire_time)))
FROM db_datastory_trident.all_temp_tags
WHERE entity_type=497
AND label_date='2021-01-18'
AND label_id=3275
GROUP BY entity_id;

SELECT entity_id,
       label_val,
       update_time
FROM tmp_trident_es_543_1_label_3275 tmp 
LATERAL VIEW explode(tree_1_label) AS label_val 
LATERAL VIEW explode(tree_1_label_update_time) AS update_time 
LIMIT 10;



```