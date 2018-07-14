# Gateway installation procedure for Autonomous APIPCS (English)

## Notes

- A virtual or physical node where gateway software appliance is installed has been already ready.
- If installing gateway software appliance to Windows environment, Configuration of OpenSSL and Python is required in advance.
- This procedure is based on the following environment. You should configure properly based on your environment.

| Items | Conditions |
|:--|:--|
| Gateway Node | VM.Standard2.1 (OCI) |
| Installation and execution user | oracle |
| JDK (JAVA_HOME) | /u01/java |
| Directory where Gateway Installer is extracted | /u01/gwinst |
| Gateway Install Directory Gateway | /u01/apipcs/gw/install |

## Documents

Official document is available in the following URL.
> Installing a Gateway Node <br/>
> [https://docs.oracle.com/en/cloud/paas/api-platform-cloud/apfad/installing-gateway-node.html#GUID-6848A2B0-03D0-435B-B867-5D9FD80E595B](https://docs.oracle.com/en/cloud/paas/api-platform-cloud/apfad/installing-gateway-node.html#GUID-6848A2B0-03D0-435B-B867-5D9FD80E595B)

## Pre-configuration

1. Add user (oracle) to the node.
2. Create directory (/u01) and mount volume if needed.
3. Configuration to resolve host name.
    - Configure /etc/hosts, /etc/sysconfig/network, or /etc/hostname
4. If needed, create Swapfile (In case of OCI-C, Swapfile creation is required. In case of OCI, swapfile is already configured.)
    ```bash
    # Create 4GB Swapfile
    dd if=/dev/zero of=/swapfile bs=1024K count=4096
    mkswap /swapfile
    swapon /swapfile
    # Add swapfile entry to /etc/fstab to mount the swapfile automatically.
    ```
5. If using OCI, firewalld is active by default. You have to configure firewalld to permit inbound access.
    ```bash
    sudo firewall-cmd --add-port=9022/tcp --permanent
    sudo firewall-cmd --add-port=8011/tcp --permanent
    sudo firewall-cmd --reload
    ```

## Transfer, extract, and configure dependencies

1. Transfer the following dependencies to /tmp as 'opc'.
    - Gateway Installer
    - Server JRE
2. Extract transferred dependencies as 'oracle'
    - Gateway Installer：/u01/gwinst/
    - JDK : /u01/java
3. Edit ~oracle/.bash_profile to add environment variables
    ```bash
    export JAVA_HOME=/u01/java
    export PATH=$JAVA_HOME/bin:$PATH
    ```
4. Reflect configured environment variables to the current session.
    ```bash
    . ~oracle/.bash_profile
    ```
5. Modify random source of JDK（If installing JDK as 'root', configuration is required as 'root'.）
    - Open file $JAVA_HOME/jre/lib/security/java.security
    - Modify 'securerandom.source' from 'securerandom.source=file:/dev/random' to 'securerandom.source=file:/dev/urandom'
    ```bash
    securerandom.source=file:/dev/urandom
    ```

## Install Gateway software appliance

1. Configure gateway-props.json. Configuration wizard in management service is helpful.
    ```json
    {
        "logicalGatewayId" : "100",
        "logicalGateway": "LGW01",
        "managementServiceUrl" : "https://xxxxx.apiplatform.ocp.oraclecloud.com:443",
        "idcsUrl" : "https://idcs-xxxxx.identity.oraclecloud.com/oauth2/v1/token",
        "requestScope" : "https://xxxxx.apiplatform.ocp.oraclecloud.com:443.apiplatform",
        "gatewayNodeName" : "LGW01 Node 1",
        "gatewayNodeDescription" : "LGW01 Node 1",
        "listenIpAddress": "yyy.yyy.yyy.yyy",
        "publishAddress": "xxx.xxx.xxx.xxx",
        "nodeInstallDir" : "/u01/apipcs",
        "managementServiceConnectionProxy": [],
        "nodeProxy": [],
        "installationArchiveLocation" : "/u01/apipcs/archives",
        "oauthProfileLocation": "/u01/apipcs/gw/security/OAuth2TokenLocalEnforcerConfig.xml",
        "gatewayExecutionMode" : "Development",
        "heapSizeGb" : "2",
        "maximumHeapSizeGb" : "4",
        "gatewayMServerPort" : "8011",
        "gatewayMServerSSLPort" : "9022",
        "nodeManagerPort" : "5556",
        "coherencePort" : "8088",
        "gatewayDBPort" : "1527",
        "gatewayAdminServerPort" : "8001",
        "gatewayAdminServerSSLPort" : "9021"
    }
    ```
2. Install, create WebLogic Domain, and start gateway
    ```bash
    # Without any action parameters, you can install and configure gateway. This is the same action as 'install-configure'.
    APIGateway -f gateway-props.json

    # If you'd like to install, configure, and start gateway, you should specify 'install-configure-start' as an action parameter.
    APIGateway -f gateway-props.json -a install-configure-start

    # If you'd like to only start gateway, you should specify 'start' as an action parameter.
    APIGateway -f gateway-props.json -a start
    ```
    The following questions are asked when doing actions listed above.
    - Gateway Node administration username and password (Administrator user and password for WebLogic Server which hosts gateway.)

## Create Logical Gateway

You can create logical gateway on management service, of course.

1. Create logical gateway（This operation is not required if the gateway node is added to the existing logical gateway.）
    ```bash
    APIGateway -f gateway-props.json -a creategateway
    ```
    The following questions are asked when doing actions listed above.
    - Gateway Node administration username and password (Administrator user and password for WebLogic Server which hosts gateway.)
    - Username and password of Gateway Manager (Users who manage specified logical gateway.)
    - Client id and Client secret of APIPCS instance.

    Client Id and Client Secret for APIPCS instance are listed in IDCS.
    - Client ID and Client Secret is created when APIPCS instance was created.
    - You can access these information from IDCS console.
    > IDCS console > APICSAUTO_(instance name) > Configuration > General Information

![IDCS01](https://raw.githubusercontent.com/anishi1222/APIPCS/images/Gateway/IDCS-image01.png)

## Add gateway node to logical gateway

1. Add the gateway node to logical gateway specified in gateway-props.json
    ```bash
    APIGateway -f gateway-props.json -a join
    ```
    The following questions are asked when doing actions listed above.
    - Gateway Node administration username and password (Administrator user and password for WebLogic Server which hosts gateway.)
    - Username and password of Gateway Manager (User who manages specified logical gateway.)
    - Client id and Client secret of APIPCS instance for Gateway Manager.
    - Username and password of Gateway Runtime User (This user is used when the gateway node communicates with management service.)
    - Client id and Client secret of APIPCS instance for Gateway Runtime User.

    Client Id and Client Secret for APIPCS instance are listed in IDCS.
    - Client ID and Client Secret is created when APIPCS instance was created.
    - You can access these information from IDCS console.
    > IDCS console > APICSAUTO_(instance name) > Configuration > General Information
