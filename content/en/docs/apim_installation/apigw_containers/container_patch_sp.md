---
title: Apply a patch, update, or service pack
linkTitle: Apply a patch or update
weight: 108
date: 2019-09-18T00:00:00.000Z
description: >-
  Apply a patch, update, or service pack (SP) to an API Gateway or API Manager
  container deployment.
---

In a container deployment, a patch or update is rolled out using an orchestration tool (for example, Kubernetes or OpenShift) after new Docker images containing the patch or update are pushed to the Docker registry. This enables you to perform a rolling zero downtime update of services.

## Apply a patch

To apply a patch, follow these steps:

{{< alert title="Note" color="primary" >}}Before you start, check the release notes of this patch for any specific instructions.{{< /alert >}}

1. Download the patch from Axway Support at [Axway Support](https://support.axway.com/).
2. Create a merge directory to contain the patch files and any custom configuration (for example, `/tmp/apigateway`). The merge directory must be called `apigateway` and must have the same directory structure as the `apigateway` directory of an API Gateway installation.
3. Unzip and extract the patch into the merge directory.
4. Add any custom configuration to the merge directory. For example, to add a custom `envSettings.props` file to your image, copy `envSettings.props` to `/tmp/apigateway/conf/`.
5. Create new Admin Node Manager and API Gateway images using the `--merge-dir` option to specify the merge directory containing the patch files and custom configuration.

    ```
    cd emt_containers-<version>
    ./build_anm_image.py --domain-cert=certs/mydomain/mydomain-cert.pem --domain-key=certs/mydomain/mydomain-key.pem --domain-key-pass-file=/tmp/pass.txt --anm-username=gwadmin --anm-pass-file=/tmp/gwadminpass.txt --merge-dir=/tmp/apigateway
    ./build_gw_image.py --license=/tmp/api_gw.lic --domain-cert=certs/mydomain/mydomain-cert.pem --domain-key=certs/mydomain/mydomain-key.pem --domain-key-pass-file=/tmp/pass.txt --merge-dir=/tmp/apigateway
    ```

## Apply an update

Use these instructions to apply an update (7.7.20200130 or later) or a service pack (7.7 SP1 or 7.7 SP2) to API Gateway version 7.7.

To apply an update or service pack, follow these steps:

{{< alert title="Note" color="primary" >}}Before you start, check the release notes of the update or service pack for any specific instructions.{{< /alert >}}

1. Download the latest API Gateway 7.7 Linux installer (which includes the update or service pack) from [Axway Support](https://support.axway.com/).
2. Create a new base image using the `--installer` option to build the image from the downloaded API Gateway installer.

    ```
    cd emt_containers-<version>
    ./build_base_image.py --installer=apigw-new-installer.run --os=centos7
    ```

3. Create a merge directory to contain any custom configuration (for example, `/tmp/apigateway`). The merge directory must be called `apigateway` and must have the same directory structure as the `apigateway` directory of an API Gateway installation.
4. Add any custom configuration to the merge directory. For example, to add a custom `envSettings.props` file to your image, copy `envSettings.props` to `/tmp/apigateway/conf/`.
5. Create new Admin Node Manager and API Gateway images using the `--merge-dir` option to specify the merge directory containing the custom configuration.

    ```
    cd emt_containers-<version>
    ./build_anm_image.py --domain-cert=certs/mydomain/mydomain-cert.pem --domain-key=certs/mydomain/mydomain-key.pem --domain-key-pass-file=/tmp/pass.txt --anm-username=gwadmin --anm-pass-file=/tmp/gwadminpass.txt --merge-dir=/tmp/apigateway
    ./build_gw_image.py --license=/tmp/api_gw.lic --domain-cert=certs/mydomain/mydomain-cert.pem --domain-key=certs/mydomain/ mydomain-key.pem --domain-key-pass-file=/tmp/pass.txt --merge-dir=/tmp/apigateway
    ```

## Apply a configuration update

Certain API Gateway, API Manager and Admin Node Manager configurations can be deployed to containerized environments without the need to re-build the underlying docker images. Deployment of selected configuration types is controlled via the Config Deployer API. This API allows clients to upload new configurations to a shared persisent volume which is accessible by all containers in the environment.

![Config Deployer Overview.](/Images/ContainerGuide/config_deployer.png)

Updatable configurations include:

| Configuration Type                        | Context                                                        | Description  |
| ----------------------------------------- | -------------------------------------------------------------- |------------- |
| gw-fed                                    | API Gateway                                                    | Configuration data for API Gateway stored as a federated entity store |
| gw-jvm.xml                                | API Gateway                                                    | Configuration of Java system properties and modification API Gateway behaviour |
| gw-envSettings.props                      | API Gateway                                                    | Environment settings for API Gateway instances, such as host and port information. |
| anm-fed                                   | Admin Node Manager                                             | Configuration data for Node Manager stored as a federated entity store |

Configuration of the above updateable congifiguration assests is managaged via `apigateway/system/conf/configDeployer.yaml`. This configuration has the following properties:

| Property                                  | Required                                                       | Description  |
| ----------------------------------------- | -------------------------------------------------------------- |------------- |
| configDeployer                            | Yes                                                            | The list of different configuration file types and their associated properties that can be deployed using the Config Deployer API. |
| type                                      | Yes                                                            | Specifies the type of the configuration file. Valid types are gw-fed, gw-jvm.xml, gw-envSettings.props and anm-fed. |
| fileName                                  | Yes                                                            | Specifies the file name that the uploaded file name will be saved as. |
| archive                                   | Yes                                                            | When set to true, the Config Deployer will create an archive directory of the previous configurations files saved to this target directory.  Valid options are true and false. |
| explode                                   | Yes                                                            | Allows for the client to indicate that the configuration file must be exploded once it has been saved to the target directory. Valid options are true and false. |
| archiveDir                                | No                                                             | Specifies the archive directory to save the configuration file to. |
| targetDir                                 | Yes                                                            | Specifies the target directory to save the configuration file to. |

To apply changes to the `apigateway/system/conf/configDeployer.yaml`, [create and start a new Admin Node Manager Docker container](/docs/apim_installation/apigw_containers/docker_script_baseimage) and ensure that the configDeployer.yaml is present in the merge directory  `--merge-dir`.

### Deploy a configuration update

The Config Deployer API is used to deploy a configuration update. This API allows you to upload a configuration asset to the configured target directory which is visable to all containers in the environment. For example, to upload an updated envSettings.props, the following request is used:

```
curl -k -i --location --request POST 'https://anm:8090/api/configuration/configDeploy' \
--header 'Authorization: Basic <Authorization Credentials>' \
--form 'file=@"/home/user/Desktop/envSettings.props"' \
--form 'type="gw-envSettings.props"'
```

A 204 response code indicates that the requests was successful.

The Config Deployer API allows for configuration assests to be depoyed to a configurable sub directory on the target directory:

```
curl -k -i --location --request POST 'https://anm:8090/api/configuration/configDeploy' \
--header 'Authorization: Basic <Authorization Credentials>' \
--form 'file=@"/home/user/Desktop/envSettings.props"' \
--form 'type="gw-envSettings.props"' \
--form 'subDir="<your sub directory>"'
```

Once the configuration asset is successfully deployed onto the persistent volume, the client instances (e.g. API Gateway) must be restarted. For a EMT managed environment, the following command can be used:

```
kubectl -n service rollout restart deployment <name>
```
