# OpenSource Tutorial 설치 시나리오

> 목차

[TOC]

## 0. Dcos cli 설치

> OS X

```bash
[ -d /usr/local/bin ] || sudo mkdir -p /usr/local/bin && 
curl https://downloads.dcos.io/binaries/cli/darwin/x86-64/dcos-1.13/dcos -o dcos && 
sudo mv dcos /usr/local/bin && 
sudo chmod +x /usr/local/bin/dcos && 
dcos cluster setup http://192.168.0.240 && 
dcos
```

> Linux

```bash
[ -d /usr/local/bin ] || sudo mkdir -p /usr/local/bin && 
curl https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.13/dcos -o dcos && 
sudo mv dcos /usr/local/bin && 
sudo chmod +x /usr/local/bin/dcos && 
dcos cluster setup http://192.168.0.240 &&
dcos
```

> login

```bash
ID : admin
PW : admin1234
```



## 1.   Kafka 설치

> write kafka-custom.json

```json
{
    "service": {
        "name": "kafka-dev",
        "placement_strategy": "NODE",
        "placement_constraint": "[[\"hostname\",\"MAX_PER\",\"2\"]]"
    },
    "brokers": {
        "count": 3,
        "kill_grace_period": 30
    },
    "kafka": {
        "delete_topic_enable": true,
        "log_retention_hours": 128
    }
}
```



> kafka install

```bash
dcos package install --options=kafka-custom.json --package-version=2.5.0-2.1.0 kafka
```



## 2.   Dcos monitoring service (Grafana)

> write monitoring-custom.json

```json
{
    "service": {
        "name": "grafana-monitoring"
        }
}
```



> install monitoring

```bash
dcos package install --options=monitoring-custom.json dcos-monitoring
```



## 3.   Metricbeat

> write metricbeat.json

```json
{
  "id": "/metricbeat-kbsys",
  "acceptedResourceRoles": [
    "*"
  ],
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "constraints": [
    [
      "hostname",
      "UNIQUE"
    ]
  ],
  "container": {
    "portMappings": [],
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/var/run/docker.sock",
        "hostPath": "/var/run/docker.sock",
        "mode": "RO"
      },
      {
        "containerPath": "/hostfs/sys/fs/cgroup",
        "hostPath": "/sys/fs/cgroup",
        "mode": "RO"
      },
      {
        "containerPath": "/hostfs/proc",
        "hostPath": "/proc",
        "mode": "RO"
      },
      {
        "containerPath": "/hostfs",
        "hostPath": "/",
        "mode": "RO"
      },
      {
        "containerPath": "/usr/share/metricbeat/metricbeat.yml",
        "hostPath": "/data/mettest/metricbeat.docker.yml",
        "mode": "RO"
      }
    ],
    "docker": {
      "image": "dmkim104/metricbeat-kbsys:v4",
      "forcePullImage": true,
      "privileged": false,
      "parameters": []
    }
  },
  "cpus": 0.1,
  "disk": 0,
  "instances": 3,
  "maxLaunchDelaySeconds": 300,
  "mem": 1024,
  "gpus": 0,
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 1
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "healthChecks": [],
  "fetch": []
}
```



> install metricbeat

```bash
dcos marathon app add metricbeat.json
```



## 4.   Elasticsearch

> write elasticsearch.json

```json
{
  "env": {
    "cluster.name": "docker-cluster",
    "node.name": "elastic01",
    "TZ": "Asia/Seoul",
    "discovery.type": "single-node",
    "ES_JAVA_OPTS": "-Xms4g -Xmx4g",
    "bootstrap.memory_lock": "true"
  },
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/log-test/elasticsearch",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.0.247"
    ]
  ],
  "container": {
    "portMappings": [
      {
        "containerPort": 9200,
        "hostPort": 0,
        "labels": {
          "VIP_0": "log-elastic:9200"
        },
        "protocol": "tcp",
        "servicePort": 10141,
        "name": "test9200"
      },
      {
        "containerPort": 9300,
        "hostPort": 0,
        "labels": {
          "VIP_1": "/log-test/elasticsearch:9300"
        },
        "protocol": "tcp",
        "servicePort": 10142,
        "name": "test9300"
      }
    ],
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/usr/share/elasticsearch",
        "hostPath": "/data/elasticsearch",
        "mode": "RW"
      }
    ],
    "docker": {
      "image": "docker.elastic.co/elasticsearch/elasticsearch:6.7.2",
      "forcePullImage": true,
      "privileged": false,
      "parameters": [
        {
          "key": "ulimit",
          "value": "memlock=-1:-1"
        }
      ]
    }
  },
  "cpus": 2,
  "disk": 0,
  "healthChecks": [
    {
      "gracePeriodSeconds": 300,
      "intervalSeconds": 60,
      "maxConsecutiveFailures": 3,
      "portIndex": 0,
      "timeoutSeconds": 20,
      "delaySeconds": 15,
      "protocol": "MESOS_HTTP",
      "path": "/",
      "ipProtocol": "IPv4"
    }
  ],
  "instances": 1,
  "maxLaunchDelaySeconds": 3600,
  "mem": 8192,
  "gpus": 0,
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 1
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "fetch": []
}
```



> install elasticsearch

```bash
dcos marathon app add elasticsearch.json
```



## 5. logstash

> write logstash.json

```json
{
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/log-test/logstash",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.0.248"
    ]
  ],
  "container": {
    "portMappings": [
      {
        "containerPort": 9600,
        "hostPort": 0,
        "labels": {
          "VIP_0": "/log-test/logstash:9600"
        },
        "protocol": "tcp",
        "servicePort": 10156,
        "name": "logstash"
      }
    ],
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/usr/share/logstash",
        "hostPath": "/data/logstash",
        "mode": "RW"
      }
    ],
    "docker": {
      "image": "docker.elastic.co/logstash/logstash:6.7.2",
      "forcePullImage": true,
      "privileged": false,
      "parameters": []
    }
  },
  "cpus": 1,
  "disk": 0,
  "healthChecks": [
    {
      "gracePeriodSeconds": 300,
      "intervalSeconds": 60,
      "maxConsecutiveFailures": 3,
      "portIndex": 0,
      "timeoutSeconds": 20,
      "delaySeconds": 15,
      "protocol": "MESOS_HTTP",
      "path": "/",
      "ipProtocol": "IPv4"
    }
  ],
  "instances": 1,
  "maxLaunchDelaySeconds": 3600,
  "mem": 1024,
  "gpus": 0,
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 1
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "fetch": []
}
```



> install logstash

```bash
dcos marathon app add logstash.json
```



## 5.   Kibana

> write kibana.json

```json
{
  "env": {
    "SERVER_NAME": "kibanatest",
    "ELASTICSEARCH_HOSTS": "http://192.168.0.249:10141",
    "TZ": "Asia/Seoul"
  },
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/log-test/kibana-view",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.0.248"
    ]
  ],
  "container": {
    "portMappings": [
      {
        "containerPort": 5601,
        "hostPort": 0,
        "labels": {
          "VIP_0": "/log-test/kibana-view:5601"
        },
        "protocol": "tcp",
        "servicePort": 10133,
        "name": "5601"
      }
    ],
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/usr/share/kibana",
        "hostPath": "/data/kibana",
        "mode": "RW"
      }
    ],
    "docker": {
      "image": "docker.elastic.co/kibana/kibana:6.7.2",
      "forcePullImage": false,
      "privileged": false,
      "parameters": []
    }
  },
  "cpus": 1,
  "disk": 0,
  "healthChecks": [
    {
      "gracePeriodSeconds": 300,
      "intervalSeconds": 60,
      "maxConsecutiveFailures": 3,
      "portIndex": 0,
      "timeoutSeconds": 20,
      "delaySeconds": 15,
      "protocol": "MESOS_HTTP",
      "path": "/",
      "ipProtocol": "IPv4"
    }
  ],
  "instances": 1,
  "maxLaunchDelaySeconds": 3600,
  "mem": 1024,
  "gpus": 0,
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 1
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "fetch": []
}
```



> install kibana.json

```
dcos marathon app add kibana.json
```



## URLS

1. DC/OS

   ```url
   http://192.168.0.240
   ```

2. dcos monitoring service

   ```url
   http://192.168.0.240/service/grafana-monitoring/grafana
   ```

3. elasticsearch

   ```url
   http://192.168.0.249:10141/_cat/indices
   http://192.168.0.249:10141/_cat/metricbeat-kafka-2019-11-[날짜]
   ```

4. kibana

   ```url
   http://192.168.0.249:10133
   ```

   