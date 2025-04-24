---
layout: post
title:  Spring Cloud Azure Key Vault指南
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Azure Key Vault
---

## 1. 概述

在本教程中，我们将探讨云原生开发的基本原理以及使用Spring Cloud Azure Key Vault的好处。

## 2. 什么是Spring Cloud Azure？

**Spring Cloud Azure是一套全面的库和工具，专门设计用于促进Spring应用程序和Microsoft Azure服务之间的集成**。

虽然已经可以将[Java应用程序](https://www.baeldung.com/spring-boot-azure)与[Azure SDK](https://learn.microsoft.com/en-us/azure/developer/java/sdk/get-started)集成，但Spring Cloud Azure的引入将这种集成提升到了一个全新的水平。

通过利用Spring Cloud Azure提供的强大的API集，我们可以方便地与各种Azure服务(例如[Azure存储](https://learn.microsoft.com/en-us/azure/storage/common/storage-introduction)、[Cosmos DB](https://azure.microsoft.com/en-us/products/cosmos-db)等)进行交互。

**它简化了开发过程并增强了应用程序的整体安全性和性能**。

Spring Cloud Azure提供了多个模块，用于将我们的应用程序与最相关的Azure服务集成，让我们看几个例子：

- 配置管理：通过[spring-cloud-azure-starter-appconfiguration](https://mvnrepository.com/artifact/com.azure.spring/spring-cloud-azure-starter-appconfiguration)，我们可以轻松集成[Azure配置管理](https://azure.microsoft.com/en-us/products/app-configuration)，这是一项用于管理和存储部署在Azure上的应用程序的配置的服务。
- 存储：通过[spring-cloud-azure-starter-storage](https://mvnrepository.com/artifact/com.microsoft.azure/spring-cloud-azure-starter-storage)，我们可以无缝使用基于云的存储解决方案[Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/storage-introduction)来存储和管理各种类型的数据，包括非结构化、结构化和半结构化数据。
- Cosmos DB：借助[spring-cloud-azure-starter-cosmos](https://mvnrepository.com/artifact/com.azure.spring/spring-cloud-azure-starter-cosmos)，可以顺利集成多模态数据库服务[Azure Cosmos DB](https://azure.microsoft.com/en-us/products/cosmos-db/)。
- 服务总线：通过[spring-cloud-azure-starter-servicebus](https://mvnrepository.com/artifact/com.microsoft.azure/spring-cloud-starter-azure-servicebus/1.2.8)，我们可以轻松集成[Azure服务总线](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)，这是一种消息服务，可实现分布式应用程序和服务之间的解耦通信。
- Active Directory：借助[spring-cloud-azure-starter-active-directory](https://mvnrepository.com/artifact/com.azure.spring/spring-cloud-azure-starter-active-directory/5.0.0)，可以轻松实现使用[Azure Active Directory](https://azure.microsoft.com/en-us/products/active-directory)(一种提供集中身份和访问管理系统的服务)。

我们可以在[这里](https://learn.microsoft.com/en-us/azure/developer/java/spring-framework/developer-guide-overview)找到可用模块的完整列表。

## 3. 项目设置

要开始使用Azure云服务，第一步是注册[Azure订阅](https://techcommunity.microsoft.com/t5/azure/understanding-azure-account-subscription-and-directory/m-p/34800)。

**订阅完成后，让我们[安装Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)**，此命令行界面工具允许我们从本地计算机与Azure服务进行交互。

接下来，让我们打开命令提示符并运行命令：
```shell
> az login
```

登录后，我们将为订阅创建一个新的资源组：
```shell
>az group create --name spring_cloud_azure --location eastus
```

**除了通过命令行创建资源组之外，我们还可以使用Web浏览器中的[Azure门户](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal#create-resource-groups)创建新订阅**，这为管理我们的Azure资源和订阅提供了直观的界面。

接下来，我们将配置IDE。在本教程中，我们将使用IntelliJ作为IDE。

**[Azure Toolkit](https://learn.microsoft.com/en-us/azure/developer/java/toolkit-for-intellij/)是进行Azure相关开发时非常有用的工具包**，它提供专门设计的工具和资源，帮助开发人员在Azure平台上构建和管理应用程序。

因此，让我们在IDE上[安装此插件](https://learn.microsoft.com/en-us/azure/developer/java/toolkit-for-intellij/install-toolkit)，然后转到Tools > Azure > Login，这将提示我们输入Azure凭据以验证我们对平台的访问。

## 4. 集成

我们已经完成将Spring应用程序与Azure服务集成的必要准备。

在本教程中，我们将利用官方Azure SDK for Java和专用的Spring Cloud模块将Azure Key Vault服务集成到我们的应用程序中。

### 4.1 Azure Key Vault

**[Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault)是一种基于云的强大服务，可提供安全可靠的方式来存储和管理敏感数据，包括加密密钥、机密和证书**。

它可以充当应用程序的外部配置源，我们不必将敏感信息定义为配置文件中的值，而是可以将它们定义为Azure Key Vault中的机密，然后在运行时将它们安全地注入到应用程序中。

首先，我们需要在之前创建的资源组上创建一个新的Key Vault。我们可以使用Azure CLI来执行此操作，但如果愿意，我们也可以使用[Azure门户](https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-portal)：
```shell
> az keyvault create --name new_keyvault --resource-group spring_cloud_azure --location eastus
```

创建Key Vault存储后，让我们在Key Vault存储new_keyvault中创建两个机密：
```shell
> az keyvault secret set --name my-database-secret --value my-database-secret-value --vault-name new_keyvault
```

```shell
> az keyvault secret set --name my-secret --value my-secret-value --vault-name new_keyvault
```

第一个机密具有密钥my-database-secret和值my-database-secret-value，而第二个机密具有密钥my-secret和值my-secret-value。

我们还可以在Azure门户上检查并确认这些机密的创建：

![](/assets/images/2025/springcloud/springcloudazurekeyvault01.png)

### 4.2 机密客户端

一旦我们定义了这些机密，我们就可以在我们的应用程序上定义[SecretClient](https://learn.microsoft.com/en-us/java/api/com.azure.security.keyvault.secrets.secretclient?view=azure-java-stable)。

SecretClient类提供了一个客户端接口，用于从Azure Key Vault检索和管理机密。

因此让我们定义一个接口：
```java
public interface KeyVaultClient {

    SecretClient getSecretClient();

    default KeyVaultSecret getSecret(String key) {
        KeyVaultSecret secret;
        try {
            secret = getSecretClient().getSecret(key);
        } catch (Exception ex) {
            throw new NoSuchElementException(String.format("Unable to retrieve %s secret", key), ex);
        }
        return secret;
    }
}
```

KeyVaultClient接口声明了两种方法：

- 第一个方法getSecretClient()返回SecretClient类的实例。
- 第二个方法getSecret()提供了一个默认实现，用于从机密客户端检索嵌套在对象[KeyVaultSecret](https://learn.microsoft.com/en-us/dotnet/api/azure.security.keyvault.secrets.keyvaultsecret?view=azure-dotnet)中的特定机密。

现在让我们看看定义SecretClient的两种方法，一种是使用标准Azure SDK，另一种是使用Spring Cloud模块。

### 4.3 不使用Spring Cloud Azure进行集成

在这种方法中，我们将仅使用Microsoft的[Azure SDK](https://learn.microsoft.com/en-us/java/api/overview/azure/?view=azure-java-stable)公开的API来配置SecretClient。

因此，让我们将[azure-keyvault-extensions](https://mvnrepository.com/artifact/com.microsoft.azure/azure-keyvault-extensions/1.2.6)依赖添加到pom.xml文件中：
```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-keyvault-extensions</artifactId>
    <version>1.2.6</version>
</dependency>
```

现在让我们在application.yaml文件中定义配置SecretClient所需的参数：
```yaml
azure:
    keyvault:
        vaultUrl: {$myVaultUrl}
        tenantId: {$myTenantId}
        clientId: {$myClientId}
        clientSecret: {$myClientSecret}
```

我们应该用适当的值替换所有占位符。

一种选择是将值直接硬编码到application.yaml中，但是，这种方法需要存储多个敏感数据，包括clientId或clientSecret，这可能会带来安全风险。

**我们不需要对这些值进行硬编码，而是可以为每个敏感数据创建一个机密，然后[使用Azure管道将其注入到我们的配置文件中](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=yaml)**。

接下来，我们创建一个KeyVaultProperties类来处理此配置：
```java
@ConfigurationProperties("azure.keyvault")
@ConstructorBinding
public class KeyVaultProperties {
    private String vaultUrl;
    private String tenantId;
    private String clientId;
    private String clientSecret;
    //Standard constructors, getters and setters
}
```

现在让我们创建客户端类：
```java
@EnableConfigurationProperties(KeyVaultProperties.class)
@Component("KeyVaultManuallyConfiguredClient")
public class KeyVaultManuallyConfiguredClient implements KeyVaultClient {

    private KeyVaultProperties keyVaultProperties;

    private SecretClient secretClient;

    @Override
    public SecretClient getSecretClient() {
        if (secretClient == null) {
            secretClient = new SecretClientBuilder()
                    .vaultUrl(keyVaultProperties.getVaultUrl())
                    .credential(new ClientSecretCredentialBuilder()
                            .tenantId(keyVaultProperties.getTenantId())
                            .clientId(keyVaultProperties.getClientId())
                            .clientSecret(keyVaultProperties.getClientSecret())
                            .build())
                    .buildClient();
        }
        return secretClient;
    }
}
```

一旦我们注入KeyVaultClient的这个实现，getSecret()默认方法就会返回手动配置的SecretClient对象。

### 4.4 与Spring Cloud Azure集成

作为此方法的一部分，我们将使用Spring Cloud Azure Key Vault设置SecretClient，并利用该框架的另一个有用功能：将机密注入属性文件。

因此，让我们将[spring-cloud-azure-starter-keyvault-secrets](https://mvnrepository.com/artifact/com.azure.spring/spring-cloud-azure-starter-keyvault-secrets)依赖添加到pom.xml文件中：
```xml
<dependency>
   <groupId>com.azure.spring</groupId>
   <artifactId>spring-cloud-azure-starter-keyvault-secrets</artifactId>
   <version>5.12.0-beta.1</version>
</dependency>
```

接下来，让我们向application.yaml添加以下属性：
```yaml
spring:
    cloud:
        azure:
            keyvault:
                secret:
                    endpoint: {$key-vault-endpoint}
```

我们应该将key-vault-endpoint占位符替换为我们存储的URI，该URI在Azure门户的Resources > {our keyvault} > Vault URI下定义。

现在让我们创建客户端类：
```java
@Component("KeyVaultAutoconfiguredClient")
public class KeyVaultAutoconfiguredClient implements KeyVaultClient {
    private final SecretClient secretClient;

    public KeyVaultAutoconfiguredClient(SecretClient secretClient) {
        this.secretClient = secretClient;
    }

    @Override
    public SecretClient getSecretClient() {
        return secretClient;
    }
}
```

一旦我们注入KeyVaultClient的这个实现，getSecret()默认方法将返回自动配置的SecretClient对象。除了端点机密之外，我们不需要在application.yaml中指定任何配置值。

Spring Cloud将自动填充所有SecretClient的凭证参数。

我们也可以在属性文件中使用Spring Cloud Azure模块注入机密，让我们在application.yaml中添加属性：
```yaml
spring:
    cloud:
        azure:
            compatibility-verifier:
                enabled: false
            keyvault:
                secret:
                    property-sources[0]:
                        name: key-vault-property-source-1
                        endpoint: https://spring-cloud-azure.vault.azure.net/
                    property-source-enabled: true
```

通过设置标志property-source-enabled，Spring Cloud Azure从keyvault-secret-property-sources[0\]中指定的Key Vault存储中注入机密。

接下来，我们可以在application.yaml中创建一个动态属性：
```yaml
database:
    secret:
        value: ${my-database-secret}
```

当应用程序启动时，Spring Cloud Azure会将${my-database-secret}占位符替换为Azure Key Vault中定义的机密my-database-secret的实际值。

### 4.5 在属性文件中注入机密

我们已经看到了两种将机密注入属性文件的方法：使用Spring Cloud Azure Key Vault或配置Azure管道。

如果我们只使用Spring Cloud Azure Key Vault，我们应该在application.yaml中对Key Vault端点进行硬编码，以在我们的属性文件上启用注入，这可能会带来安全风险。

另一方面，使用Azure管道，无需对任何值进行硬编码，管道将替换我们application.yaml中的机密。

**因此，我们应该仅使用Spring Cloud Azure Key Vault模块来获得自动配置的SecretClient和其他功能的好处，同时将机密注入我们的属性文件的任务委托给Azure管道**。

### 4.6 运行应用程序

现在让我们运行Spring Boot应用程序：
```java
@SpringBootApplication
public class Application implements CommandLineRunner {

    @Value("${database.secret.value}")
    private String mySecret;

    private final KeyVaultClient keyVaultClient;

    public Application(@Qualifier(value = "KeyVaultAutoconfiguredClient") KeyVaultAutoconfiguredClient keyVaultAutoconfiguredClient) {
        this.keyVaultClient = keyVaultAutoconfiguredClient;
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class);

    }

    @Override
    public void run(String... args) throws Exception {
        KeyVaultSecret keyVaultSecret = keyVaultClient.getSecret("my-secret");
        System.out.println("Hey, our secret is here ->" + keyVaultSecret.getValue());
        System.out.println("Hey, our secret is here from application properties file ->" + mySecret);
    }
}
```

我们的应用程序将从自动配置的客户端和启动时注入到application.yaml中的密钥检索机密，然后在控制台上显示两者。

## 5. 总结

在本文中，我们讨论了Spring Cloud与Azure的集成。

我们了解到，使用Spring Cloud Azure Key Vault而不是Microsoft提供的Azure SDK来集成Azure Key Vault和Spring应用程序会更加简单、简洁。