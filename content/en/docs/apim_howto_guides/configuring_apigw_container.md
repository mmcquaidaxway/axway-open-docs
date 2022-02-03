{
    "title": "Configure an API Gateway Docker container",
    "linkTitle": "Configure an API Gateway Docker container",
    "weight": "30",
    "date": "2022-02-01",
    "description": "A guide on how to configure an API Gateway Docker container without the need to bake into a new Docker image."
}

## Introduction

This document describes how an API Gateway, running in EMT mode, can be configured without the need to bake that configuration into a new Docker image. This process applies to API Gateway configurations only such as policy management, system property changes, environment settings, logging configurations etc.

## Configuring an API Gateway container with Docker volumes

Docker volumes facilitates mounting API Gateway configuration files to the Docker container so that they are available to the API Gateway at startup time. API Gateway configuration files are stored in the host file system and then must be mounted to the following specific mount points:

| Docker volume mount | Description |
| ------------------- | ----------- |
| /merge/fed | Mount point location for an API Gateway policy configuration stored as a deployment package (.fed file) |
| /merge/yaml | Mount point location for yaml based API Gateway policy configuration |
| /merge/apigateway | Mount point location for all other API Gateway policy configurations e.g. jvm.xml, envSettings.props etc. |
| /merge/mandatoryFiles | Mount point location for the verification of mandatory configuration files. See below (link TBC) |

The Docker volume configuration is then added to the Docker run command to start the API Gateway container with the new runtime configuration. For example:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/fed/newFed.fed:/merge/fed -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

## Sample API Gateway Docker container configurations

This section provides examples of how to configure an API Gateway Docker containeri for different configuration types.

### API Gateway policy configuration

API Gateway policy configuration stored as a deployment package (.fed file) can be added to the Docker container runtime condfiuration by adding the `fed` file to the `/merge/fed` Docker volume:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/fed/newFed.fed -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

### YAML Entity Store configuration

YAML based API Gateway policy configuration can be added to the Docker container runtime configuration by adding the source directory to the `/merge/yaml` Docker volume:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/yaml:/merge/yaml -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

### All other API Gateway configuration

All other API Gateway configuration, such as jvm.xml and envSettings.props, can be added to the Docker container runtime condfiuration via the `/merge/apigateway` Docker volume. These configurations should be stored locally in a directory structure which mirrors `apigateway` sub folder structure inside the Docker container. For example:

```
config
├── groups
│   └── emt-group
│       └── emt-service
│           └── conf
│               ├── envSettings.props
│               └── jvm.xml
└── system
    └── conf
        └── log4j2.yaml
```

The API Gateway Docker container can then be started as follows:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/config:/merge/apigateway -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

### Configuration verification

To verify that API Gateway external configuration files, mounted on Docker volumes, have been successfully found by the Docker container at runtime, add the configuration file's path to the `mandatoryConfigurationFile.yaml` and mount to the `/merge/mandatoryFiles` Docker volume. For example:

```
required:
    - groups/emt-group/emt-service/conf/envSettings.props
    - groups/emt-group/emt-service/conf/jvm.xml
    - system/conf/log4j2.yaml    
```

This configuration file lists all the external files which should be loaded into the container at runtime. If file listed is not availble to the container, the API Gateway will fail to start.

The API Gateway Docker container can then be started as follows:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/config:/merge/apigateway -v /home/user/apigw/mandatoryFiles.yaml:/merge/mandatoryFiles -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

## Further Information

For more information on how to build an API Gateway Docker image and start an API Gateway Docker container, see [Create and start API Gateway Docker container](/docs/apim_installation/apigw_containers/docker_script_gwimage).
