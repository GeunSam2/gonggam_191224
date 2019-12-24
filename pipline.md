# 공감세미나 19/12/14

[toc]

## Pipeline

### containerizing

```
git clone http://kbsys.synology.me:30000/devops/vertx-vue-chat.git
```

```bash
mvn package
```

Dockerfile-vertx-sample

```
FROM openjdk:8u232-jre

COPY ${PWD}/target/vertx-examples-0.0.1-SNAPSHOT-fat.jar /app.jar
WORKDIR /

expose 8080

ENTRYPOINT java -jar /app.jar
```

```bash
docker build -t geunsam2/vertx-sample:1 -f ./build/Dockerfile-vertx-sample .
```

```bash
docker run -d -p 8080:8080 --name vertx-sample geunsam2/vertx-example:1
```

npm

```
yarn
yarn build
```

Dockerfile-vuejs-sample

```
FROM nginx

COPY ${PWD}/src/main/frontend/dist /usr/share/nginx/html

expose 80
```

```
docker build -t geunsam2/vuejs-sample:1 -f Dockerfile-vuejs-sample .
```

```
docker run -d -p 11996:80 --name nginx 
```



### Jenkins

install jenkins

```bash
git clone https://github.com/GeunSam2/jenkins_with_docker_dcos
```

```bash
docker build -t geunsam2/jenkins_custom:1 .
```

jenkins.json

```json
{
  "env": {
    "TZ": "Asia/Seoul"
  },
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/gonggam/jenkins",
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
        "containerPort": 8080,
        "hostPort": 0,
        "protocol": "tcp",
        "servicePort": 10170,
        "name": "jenkins"
      }
    ],
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/var/jenkins_home",
        "hostPath": "/data/jenkins_home_mnt",
        "mode": "RW"
      },
      {
        "containerPath": "/var/lib/docker",
        "hostPath": "/data/jenkins_home_mnt/docker",
        "mode": "RW"
      }
    ],
    "docker": {
      "image": "geunsam2/jenkins_dcos_docker:2.190.2c1",
      "forcePullImage": true,
      "privileged": false,
      "parameters": []
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
      "path": "/login",
      "ipProtocol": "IPv4"
    }
  ],
  "instances": 1,
  "maxLaunchDelaySeconds": 3600,
  "mem": 4096,
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

```bash
dcos marathon app add jenkins.json
```



git에 Dockerfile, dcos.json

Dockerfile-vertx-sample

```bash
FROM openjdk:8u232-jre

COPY vertx-examples-0.0.1-SNAPSHOT-fat.jar /app.jar

WORKDIR /

ENTRYPOINT java -jar app.jar
```

Dockerfile-vuejs-sample

```bash
FROM nginx

COPY dist /usr/share/nginx/html

expose 80
```

dcos-vertx-sample.json

```json
{
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/gonggam/vertx-sample",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "container": {
    "portMappings": [
      {
        "containerPort": 8080,
        "hostPort": 0,
        "protocol": "tcp",
        "servicePort": 10108,
        "name": "vertx-sample"
      }
    ],
    "type": "DOCKER",
    "volumes": [],
    "docker": {
      "image": "geunsam2/vertx-sample:1",
      "forcePullImage": true,
      "privileged": false,
      "parameters": []
    }
  },
  "cpus": 0.1,
  "disk": 0,
  "instances": 1,
  "maxLaunchDelaySeconds": 300,
  "mem": 128,
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
  "fetch": [],
  "constraints": []
}
```

dcos-vuejs.json

```json
{
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/gonggam/vuejs-sample",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "container": {
    "portMappings": [
      {
        "containerPort": 80,
        "hostPort": 0,
        "protocol": "tcp",
        "servicePort": 10109,
        "name": "vuejs-sample"
      }
    ],
    "type": "DOCKER",
    "volumes": [],
    "docker": {
      "image": "geunsam2/vuejs-sample:1",
      "forcePullImage": true,
      "privileged": false,
      "parameters": []
    }
  },
  "cpus": 0.1,
  "disk": 0,
  "instances": 1,
  "maxLaunchDelaySeconds": 300,
  "mem": 128,
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
  "fetch": [],
  "constraints": []
}
```



Jenkins shell script

```bash
#########
#VARS
#########
SERVICE_N1=vertx-sample
SERVICE_N2=vuejs-sample
BUILD_IMG1=geunsam2/${SERVICE_N1}:${BUILD_NUMBER}
BUILD_IMG2=geunsam2/${SERVICE_N2}:${BUILD_NUMBER}

#########
#BUILD npm
#########
cd ${WORKSPACE}/src/main/frontend/
yarn
yarn build
cp -ra dist ${WORKSPACE}/target/
cp -ra ${WORKSPACE}/src/main/resources/pipe/* ${WORKSPACE}/target
cd ${WORKSPACE}/target

#########
#BUILD&PUSH IMAGE
#########
docker build -t ${BUILD_IMG1} -f Dockerfile-${SERVICE_N1} .
docker build -t ${BUILD_IMG2} -f Dockerfile-${SERVICE_N2} .
docker login  -u ${REPO_ID} -p ${REPO_PW}
docker push ${BUILD_IMG1}
docker push ${BUILD_IMG2}

#########
#DCOS Control
#########
/usr/local/bin/dcos cluster setup http://192.168.0.240 --no-check --username=${DCOS_ID} --password=${DCOS_PW}
/usr/local/bin/jq .container.docker.image=\"${BUILD_IMG1}\" ${WORKSPACE}/target/dcos-${SERVICE_N1}.json | /usr/local/bin/dcos marathon  app update /gonggam/${SERVICE_N1}
/usr/local/bin/jq .container.docker.image=\"${BUILD_IMG2}\" ${WORKSPACE}/target/dcos-${SERVICE_N2}.json | /usr/local/bin/dcos marathon  app update /gonggam/${SERVICE_N2}
```





## monitoring

### elk

### Elasticsearch



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
  "id": "/gonggam/elasticsearch",
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
        "name": "test9200"
      },
      {
        "containerPort": 9300,
        "hostPort": 0,
        "labels": {
          "VIP_1": "/log-test/elasticsearch:9300"
        },
        "protocol": "tcp",
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





##	Logstash

```json
{
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/gonggam/logstash",
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
          "VIP_0": "/gonggam/logstash:9600"
        },
        "protocol": "tcp",
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



## Kibana 

```json
{
  "env": {
    "SERVER_NAME": "kibanatest",
    "ELASTICSEARCH_HOSTS": "http://log-elastic.marathon.l4lb.thisdcos.directory:9200",
    "TZ": "Asia/Seoul"
  },
  "labels": {
    "HAPROXY_GROUP": "external"
  },
  "id": "/gonggam/kibana",
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
          "VIP_0": "kibana:5601"
        },
        "protocol": "tcp",
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



##	Metricbeat 

```json
{
  "labels": {
    "kibana-service-id": "metric"
  },
  "id": "/gonggam/metricbeat",
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
  "instances": 2,
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



##	Kafka 연동



### Metricbeat Output 변경

Metricbeat 서버 : 192.168.0.247, 192.168.0.248


기존 output.elasticsearch

```yaml
output.elasticsearch:
  hosts: ["log-elastic.marathon.l4lb.thisdcos.directory:9200"]
```



```yaml
output.kafka:
  hosts: ["broker.kafka-dev.l4lb.thisdcos.directory:9092"]
 
  topic: 'metric'
  partition.round_robin:
    reachable_only: false
 
  required_acks: 1
  compression: gzip
  max_message_bytes: 10000000
```



##	setup kibana dashboards



```sh
docker run \
docker.elastic.co/beats/metricbeat:6.7.2 \
setup -E setup.kibana.host=kibana.marathon.l4lb.thisdcos.directory:5601 \
-E output.elasticsearch.hosts=["log-elastic.marathon.l4lb.thisdcos.directory:9200"]
```