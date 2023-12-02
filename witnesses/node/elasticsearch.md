---
description: https://golos.id/@lex/vnutrennii-poisk-na-golose-naidyotsya-vsyo
---

# Настройка ElasticSearch

## Установка ElasticSearch ([ссылка](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html))

```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```

```
sudo apt-get update && sudo apt-get install elasticsearch
```

### Добавляем настройки в конфиг

Добавить в конфиг `/etc/elasticsearch/elasticsearch.yml`

```
network.host: 0.0.0.0
xpack.security.enabled: true
discovery.type: single-node 
http.cors.enabled : true
http.cors.allow-origin: "*"
http.cors.allow-headers: Content-Type,Authorization
```

Перезапуск для применения настроек

```
sudo service elasticsearch restart
```

### Устанавливаем пароли на доступы

```
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

В конфиг ноды позднее нужно добавить пароль заданный к роли **elastic** (для примера 123456). Перезапуск для применения настроек

```
sudo service elasticsearch restart
```

### Добавляем read-only роль `golosclient`

```
curl -u elastic:123456 -XPOST 'localhost:9200/_xpack/security/role/golosclient_readonly_role' \
-H 'Content-Type: application/json' \
-d'{"indices":[{"names":"*","privileges":["read"]}]}'
```

### Добавляем read-only пользователя `golosclient`

```
curl -u elastic:123456 -XPOST localhost:9200/_xpack/security/user/golosclient \
-H 'Content-Type: application/json' \
-d'{"roles":["golosclient_readonly_role"],"password":"golosclient"}'
```

Перезапуск для применения настроек

```
sudo service elasticsearch restart
```

## Добавляем параметры к ноде

К списку плагинов дописываем `elastic_search`

В конфиг ноды добавляем параметры:

```
elastic-search-uri = http://172.17.0.1:9200
elastic-search-login = elastic
elastic-search-password = 123456
elastic-search-versions-depth = 10
elastic-search-skip-comments-before = 2019-01-01T00:00:00
```

где `elastic-search-uri` прописан с учётом того что нода будет запускаться через Docker, а пароль к пользователю elastic заданный на [этом шаге](elasticsearch.md#ustanavlivaem-paroli-na-dostupy).

Перезапускаем ноду с её реплеем, индекс ElasticSearch должен наполняться.

## Структура и примеры

Количество элементов в индексе

```
curl -u golosclient:golosclient -XGET https://betasearch.golos.today/blog/post/_count?pretty
```

Пример запроса поста из базы

```
curl -u golosclient:golosclient -XGET https://betasearch.golos.today/blog/post/lex.vnutrennii-poisk-na-golose-naidyotsya-vsyo?pretty
```

Пример запроса статистики ElasticSearch

```
curl -u elastic:123456 -XGET "http://localhost:9200/_stats?pretty"
```

Mapping всех типов индекса blog:

```
curl -u elastic:123456 -XGET "http://localhost:9200/blog/_mapping?pretty"
```

Ответ

```
{
  "blog" : {
    "mappings" : {
      "properties" : {
        "author" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "body" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "category" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "created" : {
          "type" : "date"
        },
        "depth" : {
          "type" : "long"
        },
        "donates" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "donates_uia" : {
          "type" : "long"
        },
        "id" : {
          "type" : "long"
        },
        "json_metadata" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "net_rshares" : {
          "type" : "long"
        },
        "parent_author" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "parent_permlink" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "permlink" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "root_author" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "root_permlink" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "root_title" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "tags" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "title" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "total_votes" : {
          "type" : "long"
        }
      }
    }
  }
}
```
