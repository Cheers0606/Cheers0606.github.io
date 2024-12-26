---
title: 标签体系改造-多index方案-test
tags: ["trident","标签体系"] 
overdue: false
categories: trident 标签体系
date: 2021-01-25 14:41:32
updated: 2021-01-25 14:41:32
---

## es mapping

```bash
PUT trident_es_497_0125
{
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
      ],
      "properties": {}
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

## hive 刷数据

### info

```sql
DROP TABLE IF EXISTS db_datastory_trident.update_temp_trident_es_497_0125_info;
CREATE EXTERNAL TABLE db_datastory_trident.update_temp_trident_es_497_0125_info(user_id string,group_id string,wx_union_id string,open_ids string,user_name string,gender INT,birthday TIMESTAMP,age INT,province string,city string,district string,community string,constellation string,resident_address string,city_level string,user_source string,marital_status string,family_size INT,is_have_children INT,children_size INT,is_have_pet INT,contact_email string,contact_telephone string,contact_phone string,contact_address string,tb_phone string,wx_phone string,internet_habits string,profession string,education string,industry string,work_address string,receive_address string,family_annual_income INT,personal_annual_income INT,market_area string,is_employee INT,is_member INT,member_status string,is_downgrade INT,downgrade_cnt INT,is_activate INT,member_level string,previous_member_level string,prev_level_change_time TIMESTAMP,member_source string,member_duration INT,activate_days INT,points_balance INT,points_to_expired INT,kol_points INT,life_cycle string,previous_life_cycle string,loyalty INT,rfm string,created_time TIMESTAMP,update_time TIMESTAMP,is_first_complete_personal_data string,tenant_code string,p_date string) 
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler' 
TBLPROPERTIES(
"es.nodes"="dev1:9203,dev1:9204",
"es.write.operation"="index",
"es.update.retry.on.conflict"="10",
"es.resource"="trident_es_497_0125/info",
"es.mapping.id"="user_id",
"es.mapping.names"="user_id:user_id,group_id:group_id,wx_union_id:wx_union_id,open_ids:open_ids,user_name:user_name,gender:gender,birthday:birthday,age:age,province:province,city:city,district:district,community:community,constellation:constellation,resident_address:resident_address,city_level:city_level,user_source:user_source,marital_status:marital_status,family_size:family_size,is_have_children:is_have_children,children_size:children_size,is_have_pet:is_have_pet,contact_email:contact_email,contact_telephone:contact_telephone,contact_phone:contact_phone,contact_address:contact_address,tb_phone:tb_phone,wx_phone:wx_phone,internet_habits:internet_habits,profession:profession,education:education,industry:industry,work_address:work_address,receive_address:receive_address,family_annual_income:family_annual_income,personal_annual_income:personal_annual_income,market_area:market_area,is_employee:is_employee,is_member:is_member,member_status:member_status,is_downgrade:is_downgrade,downgrade_cnt:downgrade_cnt,is_activate:is_activate,member_level:member_level,previous_member_level:previous_member_level,prev_level_change_time:prev_level_change_time,member_source:member_source,member_duration:member_duration,activate_days:activate_days,points_balance:points_balance,points_to_expired:points_to_expired,kol_points:kol_points,life_cycle:life_cycle,previous_life_cycle:previous_life_cycle,loyalty:loyalty,rfm:rfm,created_time:created_time,update_time:update_time,is_first_complete_personal_data:is_first_complete_personal_data,tenant_code:tenant_code,p_date:p_date"
);
INSERT INTO TABLE db_datastory_trident.update_temp_trident_es_497_0125_info SELECT user_id,group_id,wx_union_id,open_ids,user_name,gender,birthday,age,province,city,district,community,constellation,resident_address,city_level,user_source,marital_status,family_size,is_have_children,children_size,is_have_pet,contact_email,contact_telephone,contact_phone,contact_address,tb_phone,wx_phone,internet_habits,profession,education,industry,work_address,receive_address,family_annual_income,personal_annual_income,market_area,is_employee,is_member,member_status,is_downgrade,downgrade_cnt,is_activate,member_level,previous_member_level,prev_level_change_time,member_source,member_duration,activate_days,points_balance,points_to_expired,kol_points,life_cycle,previous_life_cycle,loyalty,rfm,created_time,update_time,is_first_complete_personal_data,tenant_code,p_date FROM tdm_cdp.tdm_user WHERE user_id IS NOT NULL;
DROP TABLE IF EXISTS db_datastory_trident.update_temp_trident_es_497_0125_info;
```

