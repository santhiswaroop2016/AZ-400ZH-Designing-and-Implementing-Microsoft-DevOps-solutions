---
lab:
    title: '实验室：使用 Azure 实现现有 ASP.NET 应用的现代化'
    az400Module: '模块 15：使用 Docker 管理容器'
---

# 实验室：使用 Azure 实现现有 ASP.NET 应用的现代化
# 学生实验室手册

## 实验室概述

如果你决定在迁移到 Azure 的过程中对 Web 应用程序进行现代化改造，则不必完全重新构建它们。由于成本和时间限制，使用更高级的方法（如微服务）重新构建应用程序并不总是一种选择。但是，你可以通过利用容器化和 Azure PaaS 服务来实现应用的现代化改造。例如，将应用的数据层迁移到托管服务产品/服务（例如 Azure 的 SQL 数据库）可能仅限于更新连接字符串，而无需更改基础代码。

在本实验室中，你将使用 Nerd Dinner 应用程序来说明此方法。Nerd Dinner 是一个开源 ASP.NET MVC 项目。可通过 [http://www.nerddinner.com](http://www.nerddinner.com) 查看实时运行的网站。将应用程序 DB 移动到 Azure SQL 实例，并将 Docker 支持添加到应用程序以在 Azure 容器实例中运行该应用程序。

## 目标

完成本实验室后，你将能够：

- 将 LocalDB 迁移到 Azure 中的 SQL Server
- 使用 Visual Studio 中的 Docker 工具，并为应用程序添加 Docker 支持
- 将 Docker 映像发布到 Azure 容器注册表 (ACR)
- 将 Docker 映像从 ACR 推送到 Azure 容器实例 (ACI)

## 实验室持续时间

-   预计用时：**60 分钟**

## 说明

### 准备工作

#### 登录实验室虚拟机

确保已使用以下凭据登录到 Windows 10 计算机：
    
-   用户名： **Student**
-   密码： **Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge
-   可从 [Visual Studio 下载页面](https://visualstudio.microsoft.com/downloads/)获得 Visual Studio 2019 Community Edition。Visual Studio 2019 安装应包括 **“ASP.NET 和 Web 开发”**、**“Azure 开发”** 和 **“NET Core 跨平台开发”**工作负载。它已预先安装在实验室计算机上。
-   可从 [Docker 文档站点](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)获得适用于 **Windows 的 Docker**。此应用程序将作为本实验室先决条件安装。 

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括克隆 Nerd Dinner 应用程序和配置 Docker Desktop。

#### 任务 1：克隆 Nerd Dinner 应用程序

在此任务中，你将克隆 Nerd Dinner 应用程序，然后在 Visual Studio 中将其打开。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [https://github.com/spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4)。 
1.  在 [spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4) 页面上，单击 **“代码”**，然后在下拉列表中单击代码存储库的 HTTPS URL 旁边的剪贴板图标。
1.  在实验室计算机上，启动 Visual Studio，然后在启动页面上，单击 **“克隆存储库”**。
1.  在 **“克隆存储库”** 页面上的 **“存储库位置”** 文本框中，粘贴剪贴板的内容，然后单击 **“克隆”**。
1.  如果提示安装其他组件，请单击 **“安装”**。
1.  在 Visual Studio 窗口的顶部菜单中，单击 **“生成”**，然后在下拉菜单中单击 **“重新生成解决方案”**。
1.  在 Visual Studio 窗口的顶部菜单中，单击 **“IIS Express”**。这将自动打开显示 Web 应用程序的 Web 浏览器。
1.  验证应用程序是否在本地运行，然后关闭显示正在运行的应用程序的 Web 浏览器窗口。

#### 任务 2：配置 Docker Desktop 

1.  在实验室计算机上，启动 Web 浏览器，导航到 [Docker 文档站点](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)，下载适用于 Windows 的 Docker Desktop，并使用默认设置安装。
1.  在实验室计算机上，展开任务栏，右键单击 **“Docker”** 图标，然后在出现的菜单中选择 **“切换到 Windows 容器”** 选项。
1.  出现确认提示时，单击 **“切换”**。

### 练习 1：对 Nerd Dinner 应用程序进行现代化改造

在此练习中，你将使用基于 Docker 的容器化和 Azure SQL 数据库对 Nerd Dinner 应用程序进行现代化改造。

#### 任务 1：创建 Azure SQL 数据库

在此任务中，你将创建 Azure SQL 数据库。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1.  在 Azure 门户中，搜索并选择 **“SQL 数据库”** 资源类型，在 **“SQL 数据库”** 边栏选项卡上，单击 **“添加”**。
1.  在 **“创建 SQL 数据库”** 边栏选项卡的 **“基本”** 选项卡中，指定以下设置：

    | 设置 | 值 | 
    | --- | --- |
    | 订阅 | Azure 订阅的名称 |
    | 资源组 | 新资源组名称 **az400m15l01a-RG** |
    | 数据库名称 | **nerddinnerlab** | 

1.  在 **“创建 SQL 数据库”** 边栏选项卡的 **“基本”** 选项卡上，单击 **“选择服务器”** 下拉列表正下方的 **“新建”**。 
1.  在 **“新建服务器”** 边栏选项卡上，指定以下设置，并单击 **“确定”**：

    | 设置 | 值 | 
    | --- | --- |
    | 服务器名称 | 任何有效的全局唯一服务器名称 | 
    | 服务器管理员登录 | **sqluser** |
    | 密码 | **Pa55w.rd1234** |
    | 位置 | 靠近实验室位置并且可在其中预配 Azure SQL 数据库的任何 Azure 区域 |

    > **备注**： 请记录所使用的服务器名称。稍后将在本实验室用到它。

1.  返回 **“创建 SQL 数据库”** 边栏选项卡的 **“基本”** 选项卡，单击 **“计算 + 存储”** 标签旁边的 **“配置数据库”**。
1.  在 **“配置”** 边栏选项卡上，单击 **“寻找基本、标准、高级？”**，然后单击 **“应用”**。
1.  返回 **“创建 SQL 数据库”** 边栏选项卡的 **“基本”** 选项卡，单击  **“下一步: 网络 >”** ：
1.  在 **“创建 SQL 数据库”** 边栏选项卡的 **“网络”** 选项卡上，指定以下设置（将其他设置保留为默认值），然后单击 **“其他设置”**：

    | 设置 | 值 | 
    | --- | --- |
    | 连接方法 | **公共终结点** |
    | 允许 Azure 服务和资源访问服务器 | **是** |
    | 添加当前客户端 IP 地址 | **是** |

1.  在 **“创建 SQL 数据库”** 边栏选项卡的 **“其他设置”** 选项卡上，指定以下设置（将其他设置保留为默认值），然后单击 **“查看 + 创建”**：

    | 设置 | 值 | 
    | --- | --- |
    | 使用现有数据 | **无** |
    | Azure Defender 或 SQL | **以后再说** |

1.  在 **“创建 SQL 数据库”** 边栏选项卡的 **“查看 + 创建”** 选项卡中，单击 **“创建”**。 

    > **备注**： 等待部署完成。该过程需要约 5 分钟。

#### 任务 2：在 Azure 中将 LocalDB 迁移到 SQL Server

在此任务中，你需要将应用程序的 LocalDB 迁移到在上一个任务中创建的 Azure SQL 数据库。

1.  在 Visual Studio 的顶部菜单中，单击 **“查看”**，然后在下拉菜单中单击 **“SQL Server 对象资源管理器”**。
1.  在 **“SQL Server 对象资源管理器”** 窗格中，单击 **“添加服务器”** 图标，在 **“连接”** 对话框中，指定以下设置，然后单击 **“连接”** （将 **<server-name>** 替换为在上一个任务中创建的逻辑服务器的名称）：

    | 设置 | 值 | 
    | --- | --- |
    | 服务器名称 | **<server-name>.database.windows.net** |
    | 身份验证 | **SQL Server 身份验证** |
    | 用户名 | **sqluser** |
    | 密码 | **Pa55w.rd1234** |
    | 数据库名称 | **nerddinnerlab** | 

    > **备注**： 首先，将架构从 LocalDB 实例复制到新预配的 Azure SQL 数据库。

1.  在 **“SQL Server 对象资源管理器”** 窗格中，展开代表 **localDB** 实例的 localdb 节点，然后展开其 **“Databases”** 文件夹。  
1.  在数据库列表中，右键单击基于 Nerd Dinner 应用程序存储库创建的项目中的数据库，然后在出现的菜单中单击 **“架构比较”**。这将在 Visual Studio 窗口的主窗格中打开 **“SqlSchemaCompare1”** 选项卡。
1.  在 **“SqlSchemaCompare1”** 选项卡顶部的 **“选择目标”** 下拉列表中，单击 **“选择目标”**。 
1.  在 **“选择目标架构”** 对话框中，确保已选择 **“数据库”** 选项，单击 **“选择连接”**，在 **“连接”** 对话框中，选择 **“nerddinnerlab”** Azure SQL 数据库，单击 **“连接”**，然后回到 **“选择目标架构”** 对话框，单击 **“确定”**。
1.  在 Visual Studio 窗口的主窗格中，在 **“SqlSchemaCompare1”** 选项卡上，单击 **“比较”**。等待比较完成。
1.  比较完成时，在 **“SqlSchemaCompare1”** 选项卡上，单击 **“更新”**，并在出现确认提示时，单击 **“是”**。
1.  关闭 **“SqlSchemaCompare1”** 选项卡。

    > **备注**： 现在，你需要将数据从 LocalDB 实例复制到新预配的 Azure SQL 数据库。

1.  在 **“SQL Server 对象资源管理器”** 窗格中，右键单击 LocalDB 实例，然后在出现的菜单中单击 **“数据比较”**。这将启动 **“新数据比较”** 向导并打开 **“SqlDataCompare1”** 选项卡。
1.  在 **“新数据比较”** 向导的 **“选择源和目标数据库”** 页面上，在 **“目标数据库”** 部分中单击 **“选择连接”**，在 **“连接”** 对话框中，选择 **“nerddinnerlab”** Azure SQL 数据库，单击 **“连接”**，然后返回 **“选择源和目标数据库”** 页面，单击 **“下一步”**。
1.  在 **“选择要比较的表、字段和视图”** 上，选中 **“表”** 和 **“视图”** 条目旁边的复选框，然后单击 **“完成”**。
1.  在 **“SqlDataCompare1”** 选项卡的顶部菜单上，单击 **“更新目标”**，并在出现确认提示时，单击 **“是”**。
1.  关闭 **“SqlDataCompare1”** 选项卡。

    > **备注**： 为了避免代码更改，你将使用 **web.config** 转换。在此特定情况下，你将添加一个新的 **“web.release.config”** 条目。 

1.  在 Visual Studio 的 **“解决方案资源管理器”** 窗格中，展开 **“Web.config”** 条目，然后选择 **“Web.Release.config”**。这将在 Visual Studio 窗口的中心窗格中打开 **“Web.Release.config”** 文件。
1.  在 **“Web.Release.config”** 窗格上，在 `<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">` 行下方添加以下条目（将 `<server_name>` 占位符替换为你在本练习的第一个任务中创建的服务器的名称），并保存所做的更改：

    ```csharp
    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=<server_name>.database.windows.net;Initial Catalog=nerddinnerlab;Integrated Security=False;User ID=sqluser;Password=Pa55w.rd1234;Connect Timeout=30;Encrypt=True;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
    </connectionStrings>
    ```

    > **备注**： 现在，你已成功将应用程序的 LocalDB 迁移到 Azure SQL 数据库，并更新了连接字符串以反映数据库位置的更改。

#### 任务 3：使用 Visual Studio 添加 Docker 支持并在 Docker 容器内本地调试应用程序

在此任务中，你将向 Visual Studio 添加 Docker 支持，并使用该支持在 Docker 容器内本地调试应用程序。

1.  要使用 Visual Studio 实现 Docker 应用程序容器化，请在 Visual Studio 中的 **“解决方案资源管理器”** 窗口中，右键单击 **“NerdDinner”** 项目，在出现的菜单中选择 **“添加”**，然后在级联菜单中单击 **“容器业务流程协调程序支持”**。
1.  在 **“添加容器业务流程协调程序支持”** 对话框的 **“容器业务流程协调程序”** 下拉列表中，选择 **“Docker Compose”**，然后单击 **“确定”**。这将自动在 Visual Studio 窗口的主窗格中打开 **“Dockerfile”** 选项卡。
1.  如果系统提示启动 Docker Desktop，请单击 **“是”**。 

    > **备注**： Visual Studio 会自动将所需文件添加到解决方案中，包括 **docker-compose** 项目和 **Dockerfile**。它还会检查项目，以确定要用于项目的适当基础映像。对于 Nerd Dinner 解决方案，它选择使用 **microsoft/aspnet:4.8-windowsservercore-ltsc2019 基础映像：**

    ```csharp
    FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2019
    ARG 源
    WORKDIR /inetpub/wwwroot
    COPY ${source:-obj/Docker/publish} .
    ```
    > **备注**： 要在本地运行应用程序并使用 Visual Studio 在 Docker 容器中进行调试以及测试与 Azure SQL 数据库的连接，请将 **“docker-compose”** 设置为启动项目并启动它。

1.  在 Visual Studio 的 **“解决方案资源管理器”** 窗口中，右键单击 **“docker-compose”** 项目，然后在右键菜单中选择 **“设为启动项目”**。
1.  在 Visual Studio 顶部菜单中，单击 **“Docker Compose”**。

    > **备注**： Visual Studio 将下载基础映像，然后生成该映像以进行部署。生成完成后，你将看到应用程序在本地浏览器中启动。

    > **备注**： 下载可能需要花费大量时间，具体取决于可用带宽。

1.  在实验室计算机上，以管理员身份启动 **“命令提示符”**，然后在 **“管理员: C:\\windows\\system32\cmd.exe”** 中运行以下命令，以列出本地 Docker 映像，并验证列表中包含 **Dockerfile** 中引用的映像：

    ```cmd
    docker images
    ```

#### 任务 4：将 Docker 映像发布到 Azure 容器注册表

在此任务中，你将使用 Visual Studio 将在上一步中生成的 Docker 映像发布到 Azure 容器注册表。

1.  在 Visual Studio 的 **“解决方案资源管理器”** 窗口中，右键单击 **“NerdDinner”** 项目，然后在出现的菜单中单击 **“发布”**。这将启动 **“发布”** 向导。
1.  在 **“发布”** 向导的 **“如今在哪里发布?”** 页面上，确保选择 **“Azure”** 选项，然后单击 **“下一步”**。
1.  在 **“发布”** 向导的 **“你想使用哪种 Azure 服务托管应用程序?”** 页面上，选择 **“Azure 容器注册表”** 选项，然后单击 **“下一步”**。
1.  如果需要，在 **“发布”** 向导的 **“选择现有 Azure 容器注册表或创建新的 Azure 容器注册表”** 页面上，单击 **“登录”**，当系统出现提示时，使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1.  在 **“发布”** 向导的 **“选择现有 Azure 容器注册表或创建新的 Azure 容器注册表”** 页面上，单击加号。
1.  在 **“新建”** 对话框中，指定以下设置，然后单击 **“创建”**：

    | 设置 | 值 | 
    | --- | --- |
    | DNS 前缀 | 接受默认值 |
    | 订阅 | Azure 订阅的名称 |
    | 资源组 | **az400m15l01a-RG** |
    | SKU | **标准** |
    | 注册表位置 | 为部署 Azure SQL 数据库选择的同一 Azure 区域 |

1.  返回 **“发布”** 向导的 **“选择现有 Azure 容器注册表或创建新的 Azure 容器注册表”** 页，单击 **“完成”**。
1.  在 Visual Studio 界面的 **“NerdDinner”** 选项卡上，单击 **“发布”**。

    > **备注**： 等待发布操作完成。发布操作将创建 Docker 映像并将其推送到 Azure 容器注册表。

1.  发布成功后，切换到显示 Azure 门户的 Web 浏览器窗口，在 Azure 门户中搜索并选择 **“容器注册表”**，然后在 **“Azure 容器注册表”** 上单击代表示新创建的 Azure 容器注册表的条目，在其边栏选项卡上，在左侧垂直菜单的 **“服务”** 部分，单击 **“存储库”**，并验证它是否包含 **“nerddiner”** 条目。

#### 任务 5：将新的 Docker 映像从 ACR 推送到 Azure 容器实例 (ACI)

在此任务中，你将创建一个 Azure 容器实例 (ACI)，并将来自 ACR 的新上传的 Docker 映像推送到该实例。

> **备注**： 也可以将容器部署到 Azure Kubernetes 服务 (AKS) 或 Service Fabric。

> **备注**： 你将使用 Azure CLI 创建 Azure 容器实例。

1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的 **“Cloud Shell”** 图标。 
1.  如果提示选择 **“Bash”** 或 **“PowerShell”**，请选择 **“Bash”**。 

    >**备注**： 如果这是你第一次打开 **“Cloud Shell”**，会看到 **“未装载任何存储”** 消息，请选择你在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1.  从 Cloud Shell 窗格的 Bash 会话中运行以下命令，为 ACI 创建新的资源组：

    ```bash
    RESOURCEGROUPNAME1='az400m15l01a-RG'
    LOCATION=$(az group show --name $RESOURCEGROUPNAME1 --query location --output tsv)
    RESOURCEGROUPNAME2='az400m15l02a-RG'
    az group create --name $RESOURCEGROUPNAME2 --location $LOCATION
    ```

1.  从 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建 ACI 并将 NerdDinner 映像从在上一个任务中创建的 ACR 推送到其中：

    ```bash
    ACINAME='nerddinner'$RANDOM$RANDOM
    ACRNAME=$(az acr list --resource-group $RESOURCEGROUPNAME1 --query '[].name' --output tsv)
    ```

1.  从 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建具有对上一个任务中创建的 ACR 的 **“拉取”** 权限的服务主体，并将其密码存储在 shell 变量中：

    ```bash
    SPPASSWORD=$(az ad sp create-for-rbac \
    --name http://$ACRNAME-pull \
    --scopes $(az acr show --name $ACRNAME --query id --output tsv) \
    --role acrpull \
    --query password \
    --output tsv)
    ```

1.  从 Cloud Shell 窗格的 Bash 会话中，运行以下命令以检索在上一步中创建的服务主体的名称，并将其密码存储在 shell 变量中：

    ```bash  
    SPUSERNAME=$(az ad sp show --id http://$ACRNAME-pull --query appId --output tsv)
    ```

1.  从 Cloud Shell 窗格的 Bash 会话中，运行以下命令以检索包含映像的 Azure 容器注册表的名称，并将其存储在 shell 变量中：

    ```bash  
    ACRLOGINSERVER=$(az acr show --name $ACRNAME --resource-group $RESOURCEGROUPNAME1 --query "loginServer" --output tsv)
    ```

1.  从 Cloud Shell 窗格的 Bash 会话中，运行以下命令以基于 Azure 容器注册表中存储的映像创建 Azure 容器实例，你可以使用在此任务中创建的具有拉取权限的服务主体访问该实例：

    ```bash
    az container create \
    --name $ACINAME \
    --resource-group $RESOURCEGROUPNAME2 \
    --image $ACRLOGINSERVER/nerddinner:latest \
    --registry-login-server $ACRLOGINSERVER \
    --registry-username $SPUSERNAME \
    --registry-password $SPPASSWORD \
    --dns-name-label $ACINAME \
    --os-type windows \
    --query ipAddress.fqdn
    ```

    > **备注**： 请等待部署完成。该操作需要约 5 分钟。输出将包括分配给 Azure 容器实例的完全限定的 DNS 域名。

1.  在实验室计算机上，打开另一个 Web 浏览器标签页，导航到基于预配 Azure 容器实例的命令的输出标识的 IP 地址，并验证 NerdDinner 应用程序是否正在运行。

## 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**： 请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**： 该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 回顾

在本实验室中，你学习了如何用最少的代码和配置更改通过 Azure 云和 Windows 容器对现有 .NET 应用程序进行现代化改造。