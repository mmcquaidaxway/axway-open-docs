{
    "title": "Docker Volumes with API GW",
    "linkTitle": "Docker Volumes with API GW",
    "weight": "30",
    "date": "2022-02-01",
    "description": "How to configure an API Gateway Docker Container to use a Persisted Volume for API GW configuration."
}

## Introduction

The primary documentation outlining using the API GW in containers can be found here, see [Create and start API Gateway Docker container](/docs/apim_installation/apigw_containers/docker_script_gwimage). It outlines the creation of images and their deployment to containers.

This following section describes how during deployment of an image, a persisted store can be utilised to update the configuration within the container. This process applies to API Gateway configurations only such as policy management, system property changes, environment settings, logging configurations etc. This ability will allow users to update configuration without the need to rebake the Docker Image.

## Crating an API Gateway Docker Volumes

Docker Volumes facilitate mounting API Gateway configuration files to the Docker container so that they are available to the API Gateway at run time.
API Gateway configuration files are stored in the host file system. This file system can be across any hardware / cloud service offering the client utilising.

The expected mounted directory structure should be as follows:

| Docker volume mount | Description |
| ------------------- | ----------- |
| /merge/fed | Mount point location for an API Gateway policy configuration stored as a deployment package (.fed file) |
| /merge/yaml | Mount point location for yaml based API Gateway policy configuration |
| /merge/apigateway | Mount point location for all other API Gateway policy configurations e.g. jvm.xml, envSettings.props etc. |
| /merge/mandatoryFiles | Mount point location for the verification of mandatory configuration files. See below (link TBC) |

## Running an API Gateway Container with a Docker Volume

With the persisted store created as specified in the previous section the Docker volume configuration is then added to the Docker run command to start the API Gateway container with the new runtime configuration. For example:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/fed/newFed.fed:/merge/fed -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

## Configuration Verification

All Docker Images must be configured with default configuration files, image creation does not change. Only files that require updates in the container need to be mounted. To prevent default configuration in the image being loaded on startup, a mandatory file list must be maintained.

To verify that these API Gateway external configuration files, mounted on Docker volumes, have been successfully found by the Docker container at runtime, the file mandatoryConfigurationFile.yaml must be added to the /merge/mandatoryFiles Docker volume. For example:

```
required:
    - groups/emt-group/emt-service/conf/envSettings.props
    - groups/emt-group/emt-service/conf/jvm.xml
    - system/conf/log4j2.yaml
    - /merge/fed
```

The above configuration file lists the location of all the external files which should be loaded into the container at runtime. If a file listed is not available to the container, the API Gateway will fail to start.

The API Gateway Docker container can then be started as follows:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/config:/merge/apigateway -v /home/user/apigw/mandatoryFiles.yaml:/merge/mandatoryFiles -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

## Sample API Gateway Docker container configurations

This section provides examples of how to configure an API Gateway Docker container for different configuration types.

### API Gateway policy configuration

API Gateway policy configuration stored as a deployment package (.fed file) can be added to the Docker container runtime configuration by adding the `fed` file to the `/merge/fed` Docker volume:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/fed/newFed.fed -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

### YAML Entity Store configuration

YAML based API Gateway policy configuration can be added to the Docker container runtime configuration by adding the source directory to the `/merge/yaml` Docker volume:

```
docker run -d --name=apimgr --network=api-gateway-domain -p 8075:8075 -p 8065:8065 -p 8080:8080 -v /tmp/events:/opt/Axway/apigateway/events -v /home/user/apigw/yaml:/merge/yaml -e EMT_ANM_HOSTS=anm:8090 -e CASS_HOST=casshost1 -e METRICS_DB_URL=jdbc:mysql://metricsdb:3306/metrics?useSSL=false -e METRICS_DB_USERNAME=db_user1 -e METRICS_DB_PASS=my_db_pwd -e EMT_TRACE_LEVEL=DEBUG api-gateway-my-group:1.0
```

### All other API Gateway configuration

All other API Gateway configuration, such as jvm.xml and envSettings.props, can be added to the Docker container runtime configuration via the `/merge/apigateway` Docker volume. These configurations should be stored locally in a directory structure which mirrors `apigateway` sub folder structure inside the Docker container. For example:

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

## Further Information

For more information on how to build an API Gateway Docker image and start an API Gateway Docker container, see [Create and start API Gateway Docker container](/docs/apim_installation/apigw_containers/docker_script_gwimage).
