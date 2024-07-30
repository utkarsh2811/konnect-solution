# Problem

We are building a search bar that lets people do fuzzy search on different Konnect entities (services, routes, nodes). 
you're in charge of creating the backend ingest to power that service built on top of a [CDC stream](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#postgresql-create-events) generated by Debezium

We have provided a jsonl file containing some sample events that can be used to
simulate input stream.


Below are the tasks we want you to complete.

* develop a program that ingests the sample cdc events into a Kafka topic
* develop a program that persists the data from Kafka into Opensearch


## Get started

Run 

```
docker-compose up -d
```

to start a Kakfa cluster. 

The cluster is accessible locally at `localhost:9092` or `kafka:29092` for services running inside the container network.


You can also access Kafka-UI at `localhost:8080` to examine the ingested Kafka messages.

Opensearch is accessible locally at `localhost:9200` or `opensearch-node:9200` 
for services running inside the container network.

You can validate Opensearch is working by running sample queries

Insert
```
curl -X PUT localhost:9200/cdc/_doc/1 -H "Content-Type: application/json" -d '{"foo": "bar"}'
{"_index":"cdc","_id":"1","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}%
```

Search
```
curl localhost:9200/cdc/_search  | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   223  100   223    0     0  41527      0 --:--:-- --:--:-- --:--:-- 44600
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "cdc",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "foo": "bar"
                }
            }
        ]
    }
}
```

Run

```
docker-compose down
```

to tear down all the services. 

## Resources

* `stream.jsonl` contains cdc events that need to be ingested
* `docker-compose.yaml` contains the skeleton services to help you get started

## Solution

I have devised a two part solution to the problem. In my first part, I have written
a small service in Spark and Scala to send data from `stream.jsonl` file into a kafka topic. This service
is called as `spark-producer`. To consume these events from kafka, I have written
another service called as `spark-consumer` using Spark Structured Streaming and Kafka using Scala. This service is responsible for consuming
kafka events in a streaming manner and send them to opensearch cluster.

* Please run `docker-compose up -d` and verify all resources are up and running

* Please execute below cURL request from terminal/client whatever you are comfortable
to create index and mapping on OpenSearch:
```
curl -X PUT "localhost:9200/stream-events" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
  "properties": {
    "after": {
      "type": "object",
      "properties": {
        "key": {
          "type": "keyword"
        },
        "value": {
          "type": "object",
          "properties": {
            "object": {
              "type": "object",
              "properties": {
                "algorithm": {
                  "type": "keyword"
                },
                "config": {
                  "type": "object",
                  "properties": {
                    "aws": {
                      "type": "object",
                      "properties": {
                        "region": {
                          "type": "keyword"
                        },
                        "role_session_name": {
                          "type": "keyword"
                        }
                      }
                    },
                    "gcp": {
                      "type": "object",
                      "properties": {
                        "project_id": {
                          "type": "keyword"
                        }
                      }
                    }
                  }
                },
                "config_hash": {
                  "type": "keyword"
                },
                "connect_timeout": {
                  "type": "integer"
                },
                "connection_state": {
                  "type": "object",
                  "properties": {
                    "is_connected": {
                      "type": "boolean"
                    }
                  }
                },
                "created_at": {
                  "type": "long"
                },
                "data_plane_cert_id": {
                  "type": "keyword"
                },
                "description": {
                  "type": "keyword"
                },
                "enabled": {
                  "type": "boolean"
                },
                "hash_fallback": {
                  "type": "keyword"
                },
                "hash_on": {
                  "type": "keyword"
                },
                "hash_on_cookie_path": {
                  "type": "keyword"
                },
                "healthchecks": {
                  "type": "object",
                  "properties": {
                    "active": {
                      "type": "object",
                      "properties": {
                        "concurrency": {
                          "type": "integer"
                        },
                        "healthy": {
                          "type": "object",
                          "properties": {
                            "http_statuses": {
                              "type": "integer"
                            },
                            "interval": {
                              "type": "integer"
                            },
                            "successes": {
                              "type": "integer"
                            }
                          }
                        },
                        "http_path": {
                          "type": "keyword"
                        },
                        "https_verify_certificate": {
                          "type": "boolean"
                        },
                        "timeout": {
                          "type": "integer"
                        },
                        "type": {
                          "type": "keyword"
                        },
                        "unhealthy": {
                          "type": "object",
                          "properties": {
                            "http_failures": {
                              "type": "integer"
                            },
                            "http_statuses": {
                              "type": "integer"
                            },
                            "interval": {
                              "type": "integer"
                            },
                            "tcp_failures": {
                              "type": "integer"
                            },
                            "timeouts": {
                              "type": "integer"
                            }
                          }
                        }
                      }
                    },
                    "passive": {
                      "type": "object",
                      "properties": {
                        "healthy": {
                          "type": "object",
                          "properties": {
                            "http_statuses": {
                              "type": "integer"
                            },
                            "successes": {
                              "type": "integer"
                            }
                          }
                        },
                        "type": {
                          "type": "keyword"
                        },
                        "unhealthy": {
                          "type": "object",
                          "properties": {
                            "http_failures": {
                              "type": "integer"
                            },
                            "http_statuses": {
                              "type": "integer"
                            },
                            "tcp_failures": {
                              "type": "integer"
                            },
                            "timeouts": {
                              "type": "integer"
                            }
                          }
                        }
                      }
                    },
                    "threshold": {
                      "type": "integer"
                    }
                  }
                },
                "host": {
                  "type": "keyword"
                },
                "hostname": {
                  "type": "keyword"
                },
                "id": {
                  "type": "keyword"
                },
                "jwk": {
                  "type": "object",
                  "properties": {
                    "alg": {
                      "type": "keyword"
                    },
                    "d": {
                      "type": "keyword"
                    },
                    "dp": {
                      "type": "keyword"
                    },
                    "dq": {
                      "type": "keyword"
                    },
                    "e": {
                      "type": "keyword"
                    },
                    "kid": {
                      "type": "keyword"
                    },
                    "kty": {
                      "type": "keyword"
                    },
                    "n": {
                      "type": "keyword"
                    },
                    "q": {
                      "type": "keyword"
                    },
                    "qi": {
                      "type": "keyword"
                    },
                    "use": {
                      "type": "keyword"
                    }
                  }
                },
                "kid": {
                  "type": "keyword"
                },
                "labels": {
                  "type": "object",
                  "properties": {
                    "dp-group-id": {
                      "type": "keyword"
                    },
                    "managed-by": {
                      "type": "keyword"
                    },
                    "network-id": {
                      "type": "keyword"
                    },
                    "provider": {
                      "type": "keyword"
                    },
                    "region": {
                      "type": "keyword"
                    }
                  }
                },
                "last_ping": {
                  "type": "long"
                },
                "name": {
                  "type": "keyword"
                },
                "path": {
                  "type": "keyword"
                },
                "port": {
                  "type": "integer"
                },
                "prefix": {
                  "type": "keyword"
                },
                "process_conf": {
                  "type": "object",
                  "properties": {
                    "cluster_max_payload": {
                      "type": "integer"
                    },
                    "lmdb_map_size": {
                      "type": "keyword"
                    },
                    "plugins": {
                      "type": "keyword"
                    },
                    "router_flavor": {
                      "type": "keyword"
                    }
                  }
                },
                "protocol": {
                  "type": "keyword"
                },
                "read_timeout": {
                  "type": "integer"
                },
                "resource_type": {
                  "type": "keyword"
                },
                "retries": {
                  "type": "integer"
                },
                "set": {
                  "type": "object",
                  "properties": {
                    "id": {
                      "type": "keyword"
                    }
                  }
                },
                "slots": {
                  "type": "integer"
                },
                "tags": {
                  "type": "keyword"
                },
                "type": {
                  "type": "keyword"
                },
                "updated_at": {
                  "type": "long"
                },
                "use_srv_name": {
                  "type": "boolean"
                },
                "value": {
                  "type": "keyword"
                },
                "version": {
                  "type": "keyword"
                },
                "write_timeout": {
                  "type": "integer"
                }
              }
            },
            "type": {
              "type": "integer"
            }
          }
        }
      }
    },
    "before": {
      "type": "keyword"
    },
    "op": {
      "type": "keyword"
    },
    "ts_ms": {
      "type": "long"
    }
  }
}
}'
```
* please go to [spark-consumer](https://github.com/utkarsh2811/spark-consumer), clone it and run it on IntelliJ or whichever IDE you are comfortable with.
If you get some error related to java.io, 
you may need to supply few VM arguments in the run configuration of the program:
```
-Dspark.master=local[*]
--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
--add-opens=java.base/sun.util.calendar=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED
--add-opens=java.base/java.net=ALL-UNNAMED
--add-opens=java.base/java.nio=ALL-UNNAMED
--add-opens=java.base/java.io=ALL-UNNAMED
```

* please got [spark-producer](https://github.com/utkarsh2811/spark-producer), clone it and run it on IntelliJ or whichever IDE you are comfortable with.
* go to `http://localhost:5601/app/dev_tools#/console` and run below request by pasting it in the console:
```
{
  "query": {
    "fuzzy": {
      "after.value.object.algorithm": {
        "value": "rond-robi",
        "fuzziness": "4",
        "max_expansions": 40,
        "prefix_length": 0,
        "transpositions": true,
        "rewrite": "constant_score"
      }
    }
  }
}
```
* if everything was done correctly, you would get something similar to below in response:
```
"took": 52,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 25,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
      .
      .
      .
```
this indicates our fuzzy search on cdc-events are working.
