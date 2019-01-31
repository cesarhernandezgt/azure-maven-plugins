# Maven Plugin for Azure App Service

## Table of Content
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Quick Start](#quick-start)
  - [Goals](#goals)
      - [`azure-webapp:config`](#azure-webappconfig)
      - [`azure-webapp:deploy`](#azure-webappdeploy)
  - [Configuration Details](#configuration-details)
      - [`<javaVersion>`](#javaversion)
      - [`<javaWebContainer>` or `<webContainer>`](#javawebcontainer-or-webcontainer)
      - [`<resource>`](#resource)
      - [`<region>`](#region)
      - [`<pricingTier>`](#pricingtier)
  - [Samples](#samples)
  - [Feedback and Questions](#feedback-and-questions)
  - [Data and Telemetry](#data-and-telemetry)

## Overview
[![Maven Central](https://img.shields.io/maven-central/v/com.microsoft.azure/azure-webapp-maven-plugin.svg)](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.microsoft.azure%22%20AND%20a%3A%22azure-webapp-maven-plugin%22)

The Maven Plugin for Azure App Service provides seamless integration into Maven projects,
and makes it easier for developers to deploy to different kinds of Azure Web Apps:
  - [Web App on Linux](https://docs.microsoft.com/azure/app-service-web/app-service-linux-intro)
  - [Web App on Windows](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-overview)
  - [Web App for Containers](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image)


## Prerequisites

Tool | Required Version
---|---
JDK | 1.7 or above
Maven | 3.0 or above
## Quick Start
1. Make sure you have already logged in. You can use the Azure CLI 2.0 for authentication. More authentication methods can be found [here](../docs/common-configuration.md).

   - Install the Azure CLI 2.0 by following the instructions in the [Install Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli) article.
   - Run the following commands to log into your Azure subscription：
    ```
        $ az login
        $ az account set --subscription <subscription Id>
    ```
2. Create a new Azure App Service and choose Linux with built-in tomcat as the environment.
3. To use this plugin in your Maven project, add the following settings for the plugin to your `pom.xml` file:
    ```xml
    <project>
       ...
       <packaging>war</packaging>
       ...
       <build>
           <plugins>
               <plugin>
                   <groupId>com.microsoft.azure</groupId>
                   <artifactId>azure-webapp-maven-plugin</artifactId>
                   <!-- check Maven Central for the latest version -->
                   <version>1.5.1</version>
                   <configuration>
                       <schemaVersion>v2</schemaVersion>
                       <resourceGroup>your-resource-group</resourceGroup>
                       <appName>your-app-name</appName>
                       <region>westus</region>
                       <pricingTier>P1V2</pricingTier>
                       <runtime>
                           <os>linux</os>
                           <javaVersion>jre8</javaVersion>
                           <webContainer>tomcat 8.5</webContainer>
                       </runtime>
                       <deployment>
                           <resources>
                               <resource>
                                   <directory>${project.basedir}/target</directory>
                                   <includes>
                                       <include>*.war</include>
                                   </includes>
                               </resource>
                           </resources>
                       </deployment>
                   </configuration>
               </plugin>
           </plugins>
       </build>
    </project>
    ```
4. Use the following commands to deploy your project to Azure App Service.
    ```
    $ mvn azure-webapp:deploy
    ```

<a name="goals"></a>
## Goals

#### `azure-webapp:config`
-  A command line tool providing a great experience for managing webapp deploy configuration, it will rewrite your pom config based on your input. Detail about the configuation could be found in the following.

#### `azure-webapp:deploy`
- Deploy artifacts or docker container image to an Azure Web App based on your configuration.<br>If the specified Web App does not exist, it will be created.
- Configuration:
    Common configurations of all Maven Plugins for Azure can be found [here](../docs/common-configuration.md).

    This maven plugin supports two kinds of configurations V2 and V1 (deprecated). Specify the configuration
    `<schemaVersion>V2</schemaVersion>` to use the V2 configuration. Strongly suggest use v2Schema.
    You can find v1 schema [here](v1-schema.md)
    
    Property | Required | Description | Version
    ---|---|---|---
    `<region>`* | true | Specifies the region where your Web App will be hosted; the default value is **westus**. All valid regions at [Supported Regions](#region) section. | 0.1.0+
    `<resourceGroup>` | true | Azure Resource Group for your Web App. | 0.1.0+
    `<appName>` | true | The name of your Web App. | 0.1.0+
    `<pricingTier>`* | false | The pricing tier for your Web App. The default value is **P1V2**.| 0.1.0+
    `<deploymentSlot>` | false | The deployment slot to deploy your application. | 1.3.0+
    `<appServicePlanResourceGroup>` | false | The resource group of the existing App Service Plan. If not specified, the value defined in `<resourceGroup>` will be used by default. | 1.0.0+
    `<appServicePlanName>` | false | The name of the existing App Service Plan. | 1.0.0+
    `<appSettings>` | false | Specifies the application settings for your Web App. | 0.1.0+
    `<stopAppDuringDeployment>` | false | To stop the target Web App or not during deployment. This will prevent deployment failure caused by IIS locking files. | 0.1.4+

- Runtime settings

    Supported `<os>` values are *Linux*, *Windows* and *Docker*.
    Only the `jre8` is supported for `<javaVersion>` for Web App on Linux.
    If the `<webContainer>` is not configured and the `<os>` is Windows, tomcat 8.5 will be used as default value.
    But if the `<os>` is Linux, the web app will use the JavaSE as runtime.

    The runtime settings of v2 configuration could be omitted if users specify an existing web app in the configuration and just want to do the deploy directly.
    ```xml
    <configuration>
        ...
    <runtime>
        <os>Linux</os>
        <javaVersion>jre8</javaVersion>
        <webContainer></webContainer>
    </runtime>
        <!-- os -->
    <runtime>
        <os>Docker</os>
        <!-- only image is required -->
        <image>[hub-user/]repo-name[:tag]</image>
        <serverId></serverId>
        <registryUrl></registryUrl>
    </runtime>
    </configuration>
    ```
- Deployment settings

    Users don't need to care about the deployment type in v2 configuration.
    Just configure the resources to deploy to the Web App.

    It will use the zip deploy for fast and easy deploy.
    But if the artifact(s) are war package(s), war deploy will be used.
    Mix deploying war packages and other kinds of artifacts is not suggested and will cause errors.
    ```xml
    <configuration>
        ...
        <deployment>
         <resources>
          <resource>
            <directory>${project.basedir}/target</directory>
            <includes>
               <include>*.jar</include>
            </includes>
            <excludes>
               <exclude>*.xml</exclude>
            </excludes>
          </resource>
         </resources>
        </deployment>
    </configuration>
    ```

## Configuration Details
#### `<javaVersion>`
The supported values for Web App on Linux is only **jre8**.

The supported values for Web App on Windows:

 Supported Value | Description
---|---
`1.7` | Java 7, Newest minor version
`1.7.0_51` | Java 7, Update 51
`1.7.0_71` | Java 7, Update 71
`1.8` | Java 8, Newest minor version
`1.8.0_25` | Java 8, Update 25
`1.8.0_60` | Java 8, Update 60
`1.8.0_73` | Java 8, Update 73
`1.8.0_111` | Java 8, Update 111
`1.8.0_92` | Azul's Zulu OpenJDK 8, Update 92
`1.8.0_102` | Azul's Zulu OpenJDK 8, Update 102
> Note: It is recommended to ignore the minor version number so that the latest supported JVM will be used in your Web App.

#### `<javaWebContainer>` or `<webContainer>`
Supported Value | Description
---|---
`tomcat 7.0` | Newest Tomcat 7.0
`tomcat 7.0.50` | Tomcat 7.0.50
`tomcat 7.0.62` | Tomcat 7.0.62
`tomcat 8.0` | Newest Tomcat 8.0
`tomcat 8.0.23` | Tomcat 8.0.23
`tomcat 8.5` | Newest Tomcat 8.5
`tomcat 8.5.6` | Tomcat 8.5.6
`jetty 9.1` | Newest Jetty 9.1
`jetty 9.1.0.20131115` | Jetty 9.1.0.v20131115
`jetty 9.3` | Newest Jetty 9.3
`jetty 9.3.13.20161014` | Jetty 9.3.13.v20161014
`wildfly 14` | WildFly 14
> Note: It is recommended to ignore the minor version number so that the latest supported web container will be used in your Web App.

#### `<resource>`
Property | Description
---|---
`directory` | Specifies where the resources are stored.
`targetPath` | Specifies the target path where the resources will be deployed to.
`includes` | A list of patterns to include, e.g. `**/*.war`.
`excludes` | A list of patterns to exclude, e.g. `**/*.xml`.

> Note: The `<targetPath>` is relative to the `/site/wwwroot/` folder except one case: it is relative to the
`/site/wwwroot/webapps` when you deploy the war package.

#### `<region>`
All valid regions are listed as below. Read more at [Azure Region Availability](https://azure.microsoft.com/en-us/regions/services/).
- `westus`
- `westus2`
- `centralus`
- `eastus`
- `eastus2`
- `northcentralus`
- `southcentralus`
- `westcentralus`
- `canadacentral`
- `canadaeast`
- `brazilsouth`
- `northeurope`
- `westeurope`
- `uksouth`
- `ukwest`
- `eastasia`
- `southeastasia`
- `japaneast`
- `japanwest`
- `australiaeast`
- `australiasoutheast`
- `centralindia`
- `southindia`
- `westindia`
- `koreacentral`
- `koreasouth`

#### `<pricingTier>`
All valid pricing tiers are listed as below. Read more at [Azure App Service Plan Pricing](https://azure.microsoft.com/en-us/pricing/details/app-service/).
- `F1`
- `D1`
- `B1`
- `B2`
- `B3`
- `S1`
- `S2`
- `S3`
- `P1V2`
- `P2V2`
- `P3V2`

<a name="samples"></a>
## Samples
A few typical usages of Maven Plugin for Azure App Service Web Apps are listed at [Web App Samples](../docs/web-app-samples.md). You can choose one to quickly get started.


## FeedBack and Questions
If you encounter any bugs with the maven plugins, please file an issue in the [Issues](https://github.com/microsoft/azure-maven-plugins/issues) section of our GitHub repo.


## Data and Telemetry
This project collects usage data and sends it to Microsoft to help improve our products and services.
Read Microsoft's [privacy statement](https://privacy.microsoft.com/en-us/privacystatement) to learn more.
If you would like to opt out of sending telemetry data to Microsoft, you can set `allowTelemetry` to false in the plugin configuration.
Please read our [documents](https://aka.ms/azure-maven-config) to find more details.