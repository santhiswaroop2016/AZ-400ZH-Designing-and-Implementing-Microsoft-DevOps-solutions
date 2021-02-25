---
lab:
    title: '实验室：在 Azure 上使用 Puppet 部署应用'
    module: '模块 14：通过 Azure 提供的第三方基础结构即代码工具'
---

# 实验室：在 Azure 上使用 Puppet 部署应用
# 学生实验室手册

## 实验室概述

在本实验室中，我们将使用 Puppet 实验室中的 Puppet 将 PartsUnlimted MRP 应用程序（PU MRP 应用）部署到 Azure VM。 

Puppet 是一个配置管理系统，允许你通过描述基础结构即代码的状态自动化计算机的预配和配置。 

本实验室将包括以下概要步骤：

- 使用 Azure 资源管理器模板在 Azure VM 中预配 Puppet 主机和节点
- 在节点上安装 Puppet 代理
- 配置 Puppet 生产环境
- 测试生产环境配置
- 创建一个 Puppet 程序来描述 PU MRP 应用的先决条件
- 在节点上运行 Puppet 配置

## 实验室环境

实验室环境包括两个 Azure VM 和实验室计算机。Azure VM 包括：

- 将托管 PU MRP 应用的 Puppet 节点。你将在节点上执行的唯一任务是安装 Puppet 代理。Puppet 代理可以在 Linux 或 Windows 上运行。在本实验室中，我们将使用 Ubuntu Server。
- 一个 Puppet 主机，它通过 Puppet 程序管理应用于节点的配置。Puppet 主机必须在 Linux 上运行。在本实验室中，我们还将使用 Ubuntu Server。

你将使用 Azure 资源管理器模板来部署两个 Azure VM。实验室计算机将充当管理工作站，你将使用该工作站来连接和管理 Puppet 主机。 

## 目标

完成本实验室后，你将能够：

- 使用 Azure 资源管理器模板在 Azure VM 中预配 Puppet 主机和节点
- 在节点上安装 Puppet 代理
- 配置 Puppet 生产环境
- 测试生产环境配置
- 创建 Puppet 程序
- 在节点上运行 Puppet 配置

## 实验室持续时间

-   预计用时：**90 分钟**

## 说明

### 准备工作

#### 登录实验室虚拟机

确保已使用以下凭据登录到 Windows 10 计算机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

### 练习 1：使用 Puppet 将 PartsUnlimted MRP 应用程序部署到 Azure VM

在本练习中，你将使用 Puppet 将 PartsUnlimted MRP 应用程序部署到 Azure VM。

#### 任务 1：预配将用作 Puppet 主机及其节点的 Azure VM

在此任务中，你将使用 Azure 资源管理器模板部署两个 Azure VM 并对其进行配置。其中一个 VM 将充当 Puppet 主机，另一个将作为其托管节点。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1.  打开另一个浏览器标签页，然后导航到 [Azure 资源管理器模板链接](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FPuppet%2Fenv%2FPuppetPartsUnlimitedMRP.json)。这会自动重定向到 Azure 门户中的 **“自定义部署”** 边栏选项卡。

1.  在 Azure 门户中，在 **“自定义部署”** 边栏选项卡上，单击 **“编辑模板”**
1.  在 **“编辑模板”** 边栏选项卡上的模板概述窗格中，展开 **“变量”** 部分，然后单击 **“vmSize”**。
1.  在模板详细信息部分中，将 `"pmVmSize": "Standard_D2_V2",` 更改为 `"vmSize": "Standard_D2s_v3",`。
1.  在模板详细信息部分中，将 `"mrpVmSize": "Standard_A2",` 更改为 "vmSize": `"Standard_D2s_v3",` 并单击 **“保存”**。
1.  返回 **“自定义部署”** 边栏选项卡，指定以下设置（其他设置保留默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 **az400m14l03a-RG** |
    | 区域 | 可以在其中部署 Azure VM 的 Azure 区域的名称 |
    | Pm 管理员用户名 | **azureuser** |
    | Pm 管理员密码 | **Pa55w.rd1234** |
    | 公共 IP 的 Pm Dns 名称 | 由 3 到 63 个字母和数字组成的任意唯一字符串，以字母开头 |
    | Pm 控制台密码 | **Pa55w.rd1234** |
    | Mrp 管理员用户名 | **azureuser** |
    | Mrp 管理员密码 | **Pa55w.rd1234** |
    | 公共 IP 的 Mrp Dns 名称 | 由 3 到 63 个字母和数字组成的任意唯一字符串，以字母开头 |

    >**备注**： 记录在部署过程中指定的 DNS 名称。将第一个名称 (**Pm**) 分配到 Puppet 主机。将第二个名称 (**Mrp**) 分配到 Puppet 节点。 

1.  在 **“自定义部署”** 边栏选项卡上，单击 **“查看 + 创建”**，确认验证过程完成，然后单击 **“创建”**。

    >**备注**：等待部署完成。该操作需约 10 分钟。

1.  部署完成后，在 Azure 门户中，使用页面顶部的 **“搜索资源、服务和文档”** 文本框搜索并选择 **“虚拟机”** 资源，然后在 **“虚拟机”** 边栏选项卡上，选择代表新部署的 Azure VM 的条目，该 VM 将用作 Puppet 节点。
1.  在“虚拟机”边栏选项卡上，标识并记录 **“DNS 名称”** 属性的值。
1.  浏览回 **“虚拟机”** 边栏选项卡，然后选择代表托管 Puppet 主机的新部署的 Azure VM 的条目。
1.  在“虚拟机”边栏选项卡上，将鼠标指针悬停在 **“DNS 名称”** 条目上，然后将其复制到剪贴板。 
1.  打开一个新的浏览器标签页，然后导航到刚复制到剪贴板中的 DNS 名称。使用 **https** 前缀。这将显示 Puppet 主机控制台登录页面

    >**备注**：忽略任何证书错误。这在预料之中。 

1.  在显示 Puppet 主机控制台登录页面的 Web 浏览器窗口中，使用 **admin** 作为用户名并使用 **Pa55w.rd1234** 作为密码进行登录。这将显示 Puppet 配置管理控制台。

    >**备注**： 不能使用在部署过程中指定的用户名登录 Puppet 主机控制台，而必须使用内置的管理员用户帐户。

#### 任务 2：在节点 Azure VM 上安装 Puppet 代理

在此任务中，你要将第二个 Azure VM 添加为由 Puppet 主机管理的节点。 

1.  在显示 Puppet 配置管理控制台的 Web 浏览器中，在左侧的垂直菜单中，单击 **“节点”**，再单击 **“未签名证书”**，并记录允许你添加要由 Puppet Enterprise 管理的节点的命令。 

    >**备注**： 你将在此任务的稍后部分使用此命令。该命令将类似于以下格式（其中 `<Pm_Dns_Name_for_Public_IP>` 占位符代表你在上一个任务中赋予第二个 Azure VM 的 IP 地址的名称）

    ```bash
    curl -k https://<Pm_Dns_Name_for_Public_IP>.0h03b1jj0ewetnu40a35rbm0qg.bx.internal.cloudapp.net:8140/packages/current/install.bash | sudo bash
    ``` 

1.  在实验室计算机上的 Web 浏览器窗口中，导航到 [PuTTY 下载页面](https://putty.org/)，下载 PuTTY 安装程序，然后使用默认设置运行安装。
1.  安装完成后，在 **“开始”** 菜单中，展开 **“PuTTY(64 位)”** 文件夹，然后单击 **“PuTTY”** 图标以打开 **“PuTTY 配置”** 窗口。
1.  在 **“PuTTY 配置”** 窗口的 **“主机名(或 IP 地址)”** 文本框中，键入将托管在上一个任务结束时标识的 Puppet 节点的 Azure VM 的 DNS 名称，然后单击 **“打开”**。
1.  当系统出现提示时，在 **“PuTTY 安全警报”** 窗口中，单击 **“是”**。
1.  当系统出现提示时，在 Puppet 节点的 **PuTTY** 会话中，使用 **Pa55w.rd1234** 密码以 **azureuser** 身份登录。
1.  登录后，在 Puppet 节点的 PuTTY 会话中，运行你在此任务的前面部分记录的命令

    >**备注**： 等待命令在该节点上安装 Puppet 代理以及任何依赖项。该操作需要约 2 分钟。从现在开始，你只能使用 Puppet 主机来配置节点。

    >**备注**： 可以使用 Azure 市场中的 Puppet 代理扩展自动化 Azure VM 上的 Puppet 代理安装和配置。

    >**备注**： 接下来，要管理新安装的节点，你需要接受来自 Puppet 配置管理控制台的待处理请求。

1.  切换回显示 Puppet 配置管理控制台的 Web 浏览器窗口，刷新 **“未签名证书”** 页面，验证该页面是否显示待处理的未签名证书请求，然后单击 **“接受”** 以批准该请求，以便将节点放入清单。

    >**备注**： 这是对授权证书的请求，该证书将保护 Puppet 主机和节点之间的通信。

1.  在显示 Puppet 配置管理控制台的 Web 浏览器窗口中，单击 **“节点”**，并确认是否看到两个分别代表 Puppet 主机及其节点的条目。

    >**备注**： Parts Unlimited MRP 应用程序（PU MRP 应用）是 Java 应用程序。PU MRP 应用要求在配置为 Puppet 节点的 Azure VM 上安装 MongoDB 和 Apache Tomcat 并对其进行配置。但是，我们将编写一个可自动配置节点的 Puppet 程序，而不是手动安装 MongoDB 和 Tomcat 并对其进行配置。Puppet 程序存储在 Puppet 主机上的特定目录中。Puppet 程序由描述一个或多个目标节点预期状态的清单组成。清单可以使用模块，这些模块是预先打包的 Puppet 程序。用户可以创建自己的模块，也可以使用由 Puppet 实验室（称为 Forge）维护的市场中的模块。Forge 上的某些模块已得到官方支持，而其他模块是从社区上传的开源模块。Puppet 程序按环境组织，因此你可以管理适用于不同环境（例如开发、测试和生产环境）的 Puppet 程序。

    >**备注**： 在本实验室中，我们将使用新加入的节点来模拟生产环境。我们还将从 Forge 下载模块，并使用这些模块来配置节点。

#### 任务 3：配置 Puppet 生产环境

在此任务中，你将配置模拟的 Puppet 生产环境。 

>**备注**： 用于将 Puppet 部署到 Azure 的模板还在 Puppet 主机上配置了用于管理生产环境的目录。对应的目录是 **etc/puppetlabs/code/environments/production**。

>**备注**： 你将首先检查生产模块。

1.  在实验室计算机上的 **“开始”** 菜单中，展开 **“PuTTY(64 位)”** 文件夹，然后单击 **“PuTTY”** 图标以打开 **“PuTTY 配置”** 窗口。
1.  在 **“PuTTY 配置”** 窗口的 **“主机名(或 IP 地址)”** 文本框中，键入托管 Puppet 主机的 Azure VM 的 DNS 名称，然后单击 **“打开”**。
1.  当系统出现提示时，在 **“PuTTY 安全警报”** 窗口中，单击 **“是”**。
1.  当系统出现提示时，在 Puppet 主机的 **PuTTY** 会话中，使用 **Pa55w.rd1234** 密码以 **azureuser** 身份登录。
1.  成功通过身份验证后，在 Puppet 主机的 PuTTY 会话中，运行以下命令，将当前工作目录更改为生产目录 **etc/puppetlabs/code/environments/production**：

    ```bash
    cd /etc/puppetlabs/code/environments/production
    ```

1.  从 Puppet 主机的 PuTTY 会话，运行 `ls` 命令，以列出生产目录的内容。 

    >**备注**： 你将看到名为 **“manifests”** 和 **“modules”** 的目录。 **manifests** 目录包含配置的说明，我们将在本实验室的稍后部分将这些设置应用于示例节点。 **modules** 目录包含清单引用的所有模块。

    >**备注**： 接下来，你将从 Forge 安装配置示例节点所需的其他 Puppet 模块。 

1.  在 Puppet 主机的 PuTTY 会话中，运行以下命令以安装所需的模块：

    ```bash
    sudo puppet module install puppetlabs-mongodb
    sudo puppet module install puppetlabs-tomcat
    sudo puppet module install maestrodev-wget
    sudo puppet module install puppetlabs-accounts
    sudo puppet module install puppetlabs-java
    ```

    >**备注**： 官方支持 Forge 的 mongodb 和 tomcat 模块。wget 模块是一个用户模块，不受官方支持。accounts 模块为 Puppet 提供了用于在 Linux 上创建用户和组并对其进行管理的类。最后，Java 模块为 Puppet 提供了其他 Java 功能。

    >**备注**： 接下来，你将在 Puppet 主机的 **modules**  目录中创建一个名为 **mrpapp** 的自定义模块。自定义模块将配置 PU MRP 应用。 

1.  从 Puppet 主机的 PuTTY 会话，运行以下命令，将当前目录更改为 **etc/puppetlabs/code/environments/production/modules**：

    ```bash
    cd /etc/puppetlabs/code/environments/production/modules
    ```

1.  从 Puppet 主机的 PuTTY 会话，运行以下命令以创建 **mrpapp** 模块：

    ```bash
    sudo puppet module generate partsunlimited-mrpapp
    ```

    >**备注**： 这将启动一个向导，该向导在构建模块时会询问一系列问题。 

1.  要完成 **mrapp** 模块的创建，请针对每个问题按 **Enter**键（接受空白或默认值），直到向导完成。

    >**备注**： 可以使用 `ls -la` 命令来验证已创建新模块。`ls -la` 命令使用长列表格式 (`-l`) 列出目录的内容，包括隐藏文件 (`-a`)。

    >**备注**： mrpapp 模块将定义节点的配置。在 **site.pp** 文件中定义生产环境中节点的配置。**site.pp** 文件位于 **manifests** 目录中。 **pp** 文件扩展名是 **“Puppet Program”** 的首字母缩写。你将通过为节点添加配置来编辑 **site.pp** 文件。

1.  从 Puppet 主机的 PuTTY 会话，运行以下命令以在 Nano 文本编辑器中打开 **site.pp**：

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
    ```

1.  在 Nano 编辑器界面中，滚动到该文件底部，并将现有 `node default` 部分替换为以下代码：

    ```bash
    node default {
       class { 'mrpapp': }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。

    >**备注**： 默认情况下，你对  **site.pp** 文件的编辑指示 Puppet 为所有节点配置 **mrpapp** 模块。 **mrpapp** 模块（虽然当前为空）位于生产环境的 **modules** 目录中，这样 Puppet 就会知道在哪里找到该模块。

#### 任务 4：测试生产环境的配置 

在此任务中，你将测试生产环境的配置 

>**备注**： 在为示例节点完整描述 PU MRP 应用之前，我们将通过在 **mrpapp** 模块中设置虚拟文件来测试配置。如果 Puppet 成功执行并创建了虚拟文件，我们将继续进行 **mrpapp** 模块的完整配置。

>**备注**：你将首先编辑 **init.pp** 文件，该文件是 **mrpapp** 模块的入口点。该文件位于 Puppet 主机的 **mrpapp/manifests** 目录中。 

1.  从 Puppet 主机的 PuTTY 会话运行以下命令，以在 Nano 文本编辑器中打开 **init.pp** 文件：

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  在 Nano 编辑器界面中，滚动到 `class: mrpapp` 声明并对其进行修改，使其如下所示：

    ```puppet
    class mrpapp {
       file { '/tmp/dummy.txt':
          ensure => 'present',
          content => 'Puppet rules!'
       }
    }
    ```

    >**备注**： 确保按照说明删除类定义前面的注释标记 (`#`)。

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。

    >**备注**： Puppet 程序中的类不等同于面向对象的编程中的类。Puppet 程序中的类定义在节点上配置的资源。通过将 `mrpapp` 类或资源添加到 **init.pp** 文件，我们已指示 Puppet 确保路径 /tmp/dummy.txt 上存在一个内容为“Puppet rules!”的文件。随着本实验室的进行，我们将在 `mrpapp` 类中定义更多高级资源。

    >**备注**： 接下来开始执行新设置的测试。 

1.  切换到 Puppet 节点的 PuTTY 会话，然后运行以下命令以测试新配置：

    ```bash
    sudo puppet agent --test --debug
    ```

    >**备注**： 默认情况下，Puppet 代理每隔 30 分钟查询一次 Puppet 主机的配置。然后，Puppet 代理根据 Puppet 主机指定的配置测试其当前配置。如有必要，Puppet 代理会修改其配置以匹配 Puppet 主机指定的配置。你调用的命令会触发本地 Puppet 代理立即对 Puppet 主机进行配置检查。在本例中，配置要求 **tmp/dummy.txt** 文件存在，这样节点才会相应地创建文件。

1.  在 Puppet 节点的 PuTTY 会话中，运行以下命令以验证 **tmp/dummy.txt** 文件存在并列出该文件的内容：

    ```bash
    cat /tmp/dummy.txt
    ```

1.  验证“Puppet rules!”消息显示为命令的输出。 

    >**备注**： 接下来，你将纠正配置偏差。每次运行 Puppet 代理时，Puppet 都会确定环境是否处于正确的状态。如果状态不正确，Puppet 会按需重新应用相关的类。通过此过程，Puppet 可以检测配置偏差并对其进行修正。你将通过从节点删除虚拟文件 **dummy.txt** 来模拟配置偏差。 

1.  在 Puppet 节点的 PuTTY 会话中，运行以下命令以删除 **dummy.txt** 文件：

    ```bash
    sudo rm /tmp/dummy.txt
    ```

1.  在 Puppet 节点的 PuTTY 会话中，运行以下命令以触发 Puppet 代理的执行：

    ```bash
    sudo puppet agent --test
    ```

1.  在 Puppet 节点的 PuTTY 会话中，运行以下命令以验证 **tmp/dummy.txt** 文件存在并列出该文件的内容：

    ```bash
    cat /tmp/dummy.txt
    ```

#### 任务 5：创建一个 Puppet 程序来描述 PU MRP 应用的先决条件

在此任务中，你将创建一个 Puppet 程序来描述 PU MRP 应用的先决条件。

>**备注**： 我们已将节点连接到 Puppet 主机。现在，我们可以编写一个 Puppet 程序来描述 PU MRP 应用的先决条件。实际上，大型配置解决方案的各个部分通常分为多个清单或模块。将配置拆分为多个文件是模块化的一种形式，可以促进代码重用。为简单起见，在本实验室中，我们将在之前创建的 mrpapp 模块内的单个 Puppet 程序文件 **init.pp** 中描述整个配置。在此任务中，我们将逐步完成生成 **init.pp** 文件的过程。

>**备注**： 可以在 [Microsoft 的 PU MRP GitHub 存储库](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp)中找到完整的 **init.pp** 文件。可以从 GitHub 复制 **init.pp** 文件的内容，也可以在此任务中遍历其编辑步骤。

>**备注**： Tomcat 要求节点上存在 Linux 用户和组，才能访问和运行 Tomcat 服务。Tomcat 对用户和组均使用默认名称 `Tomcat`。可以通过几种方法实现此类配置，但为此，我们将在 **init.pp** 文件中使用一个单独的类。除了编辑 **init.pp** 文件外，我们还将进行以下修改。

- 编辑文件 **war.pp** 以简化 PU MRP 应用的部署。
- 在目录 **~/tomcat7/webapps** 上编辑权限，以便能够在此任务结束时提取 **war** 文件。通过编辑这些权限，可以在 Tomcat 服务随 **pp** 文件运行重启时自动提取 war 文件。

>**备注**： 在逐步完成此任务的各个部分时，我们将说明创建 Puppet 程序所需的所有操作。 

##### 第 5.1 部分：配置 MongoDB

>**备注**： 配置了 MongoDB 后，建议 Puppet 下载一个包含 PU MRP 应用程序数据库数据的 Mongo 脚本。我们会将该脚本包含在 MongoDB 设置中。要实现这些要求，请添加一个类以配置 MongoDB。

1.  在 Puppet 主机的 PuTTY 会话中，运行以下命令以在 Nano 文本编辑器中的 **mrpapp 模块**内打开 **init.pp** 文件：

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  在 Nano 编辑器界面中，滚动到该文件底部，并从新行开始附加 `configuremongodb`：

    ```puppet
    class configuremongodb {
      include wget
      class { 'mongodb': }->

      wget::fetch { 'mongorecords':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/deploy/MongoRecords.js',
        destination => '/tmp/MongoRecords.js',
        timeout => 0,
      }->
      exec { 'insertrecords':
        command => 'mongo ordering /tmp/MongoRecords.js',
        path => '/usr/bin:/usr/sbin',
        unless => 'test -f /tmp/initcomplete'
      }->
      file { '/tmp/initcomplete':
        ensure => 'present',
      }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，然后按 **Enter** 键以保存更改并使文件保持打开。 

    >**备注**： 我们来看看 `configuremongodb` 类。

    第 1 行。创建名为 `configuremongodb` 的 `class (resource)`。
    第 2 行。包含 `wget` 模块，以便可以通过 `wget` 下载文件。
    第 3 行。（从先前下载的 `mongodb` 模块）调用 `mongodb resource`。使用 Puppet Mongo DB 模块中定义的默认值安装 Mongo DB。以上就是安装 Mondo DB 所需执行的所有操作。
    第 5 行。从 `wget` 模块调用 `fetch` 资源，并调用资源 `mongorecords`。
    第 6 行。设置需要从 PU MRP GitHub 下载的文件的源。
    第 7 行。设置文件必须下载到其中的目标。
    第 10 行。使用内置的 Puppet 资源 `exec` 来执行命令。
    第 11 行。指定要执行的命令。
    第 12 行。设置命令调用的路径。
    第 13 行。使用关键字 `unless` 指定条件。只用执行此命令一次，以便在插入记录（第 15 行）后创建 `tmp` 文件一次。如果该文件存在，则不再执行该命令。

    >**备注**： 第 3、9 和 14 行的 `->` 符号是排序箭头。排序箭头指示 Puppet 在调用右侧资源之前必须先应用左侧资源。这使我们能够控制执行顺序。

##### 第 5.2 部分：配置 Java

1.  在 Nano 编辑器界面中，将以下 `configurejava` 类定义附加到 **init.pp** 文件的末尾，位于 `configuremongodb` 类定义的正下方，以设置 Java 要求：

    ```puppet
    class configurejava {
      include apt
      $packages = ['openjdk-8-jdk', 'openjdk-8-jre']

      apt::ppa { 'ppa:openjdk-r/ppa': }->
      package { $packages:
         ensure => 'installed',
      }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，然后按 **Enter** 键以保存更改并使文件保持打开。 

    >**备注**： 我们来看看 `configurejava` 类。

    第 2 行。包含 `apt` 模块，这将使我们能够配置新的**个人包档案 (PPA)**。
    第 3 行。创建一系列需要安装的包。
    第 5 行。添加 **PPA**。
    第 6-8 行。指示 Puppet 确保已安装包。Puppet 扩展数组并在数组中安装每个包。

    >**备注**： 不能使用 Puppet 包目标来安装 Java，因为它只能安装 Java 7。添加 PPA 并使用 `apt` 模块安装 Java 8。

##### 第 5.3 部分：创建用户和组

1.  在 Nano 编辑器界面中，将以下 `createuserandgroup` 类附加到 **init.pp** 文件的末尾，以指定用于配置、部署、启动和停止服务的 Linux 用户和组。

    ```puppet
    class createuserandgroup {

    group { 'tomcat':
      ensure => 'present',
      gid    => '10003',
     }

    user { 'tomcat':
      ensure           => 'present',
      gid              => '10003',
      home             => '/tomcat',
      password         => '!',
      password_max_age => '99999',
      password_min_age => '0',
      uid              => '1003',
     }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，然后按 **Enter** 键以保存更改并使文件保持打开。 

    >**备注**： 我们来看看 `createuserandgroup` 类。

    第 1 行。调用在 **init.pp** 文件开头定义的 `createuserandgroup` 类。定义该类以使用 `user` 和 `group` 资源。
    第 3-6 行。使用 `group` 命令来创建组 `Tomcat`。使用参数值 `ensure = > present` 来验证 `Tomcat` 组是否存在，如果不存在，则创建该组。然后，为该组分配一个 `gid` 值。此分配不是强制性的，但这有助于跨多台计算机管理组。
    第 8-17 行。使用 `user` 命令创建用户 `Tomcat`。使用 `ensure` 命令来验证 `Tomcat` 用户是否存在，如果不存在，则创建一个。分配 `gid` 和 `uid` 值来帮助管理用户。然后，设置 `Tomcat` 用户的主目录路径并分配用户的密码设置。为了易于使用，保留 `password` 值未指定。实际上，应该实现严格的密码策略。

    >**备注**： 可以通过之前安装在实验室 Puppet 主机上的 `puppetlabs-accounts` 模块使用命令 `user` 和 `group`。

##### 第 5.4 部分：配置 Tomcat

1.  在 Nano 编辑器界面中，将以下 `configuretomcat` 类附加到 **init.pp** 文件的末尾以配置 Tomcat：

    ```puppet
    class configuretomcat {
      class { 'tomcat': }
      require createuserandgroup

     tomcat::instance { 'default':
      catalina_home => '/var/lib/tomcat7',
      install_from_source => false,
      package_name => ['tomcat7','tomcat7-admin'],
     }->

     tomcat::config::server::tomcat_users {
     'tomcat':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     'tomcat7':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     }->

     tomcat::config::server::connector { 'tomcat7-http':
      catalina_base => '/var/lib/tomcat7',
      port => '9080',
      protocol => 'HTTP/1.1',
      connector_ensure => 'present',
      server_config => '/etc/tomcat7/server.xml',
     }->

     tomcat::service { 'default':
      use_jsvc => false,
      use_init => true,
      service_name => 'tomcat7',
     }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，然后按 **Enter** 键以保存更改并使文件保持打开。 

    >**备注**： 我们来看看 `configuretomcat` 类。

    第 1 行。创建名为 `configuretomcat` 的 `class (resource)`。
    第 2 行。从先前下载的 Tomcat 模块调用 `tomcat` 资源。
    第 3 行。我们要求 Tomcat 用户和组可用才能配置 Tomcat，因此按需标记类 `createuserandgroup`。
    第 6-9 行。从先前下载的 Tomcat 模块安装 `Tomcat` 包。包名称是 `Tomcat7`，因此在此处调用该包。指定主目录，我们将在其中放置默认安装项。然后，指定它并非来自源。从下载的包安装时，这是必需的。如果愿意，可以直接从 Web 源安装。还需安装 `tomcat7-admin` 包。我们不会在本实验室中使用 `tomcat7-admin` 包，但该包可用于提供 Tomcat Manager Web 用户界面。
    第 13-23 行。创建两个用户名，以供 Tomcat 使用。这两个用户名是在 Tomcat 应用程序中创建的，用于管理和配置 Tomcat。它们在文件 **var/lib/tomcat7/conf/tomcat-users.xml** 中创建并通过该文件进行调用。 
    第 26-31 行。配置 Tomcat 连接器，定义要使用的协议和端口号。还需确保连接器存在，以使其能够在我们设置的端口上处理请求。还需为 Puppet 指定连接器属性，以写入 Tomcat **server.xml** 文件。同样，可以在 Puppet 主机上的 **var/lib/tomcat7/conf/server.xml** 中打开文件并查看其内容。
    第 35-38 行。配置 Tomcat 服务。由于只安装了 Tomcat，因此我们将 `default` 指定为配置目标。通过设置 `use_init` 值来确保服务正在运行。然后，将 PU MRP 应用的服务连接器名称指定为 `tomcat7`。

##### 第 5.5 部分：部署 WAR 文件

>**备注**： 将 PU MRP 应用编译为 Tomcat 用于提供页面的 .war 文件。需要指定资源才能为 PU MRP 应用站点部署 .war 文件。

1.  在 Nano 编辑器界面中，将以下 `deploywar` 类附加到 **init.pp** 文件的末尾。

    ```puppet
    class deploywar {
      require configuretomcat

      tomcat::war { 'mrp.war':
        catalina_base => '/var/lib/tomcat7',
        war_source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/mrp.war',
      }

     file { '/var/lib/tomcat7/webapps/':
       path => '/var/lib/tomcat7/webapps/',
       ensure => 'directory',
       recurse => 'true',
       mode => '777',
     }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，然后按 **Enter** 键以保存更改并使文件保持打开。 

    >**备注**： 我们来看看 deploywar 类。

    第 1 行。创建名为 `deploywar` 的 `class (resource)`。
    第 2 行。指示 Puppet 在调用 `deploywar` 类之前确保 `configuretomcat` 已完成。
    第 5 行。设置目录 `catalina_base`，以便 Puppet 将 .war 部署到 Tomcat 服务。
    第 6 行。使用 Tomcat 模块的 `war` 资源从 `war_source` 部署 .war。
    第 9 行。调用在 Tomcat 模块中定义的 `file` 类。`file` 类允许在系统上配置文件。
    第 10 行。为 PU MRP 应用指定 Web 目录的路径。
    第 11 行。在应用配置之前，确保目录位于指定的路径上。
    第 12 行。指定应该递归地应用配置，因此它将影响 `~/tomcat7/webapps/` 目录中的所有文件和子目录。
    第 13 行。在 `~/tomcat7/webapps/` 目录中的所有文件和子目录上指定 `777` 权限。务必设置这些权限，以允许对目标目录内的所有内容具有读取、写入和执行权限。在本实验室中，可以设置 `777` 权限。应始终根据生产环境中的每个用户、组和访问级别设置限制性更强的权限。

    >**备注**： 通常，需要执行以下操作才能部署 .war 文件

    1. 将 .war 文件复制到 Tomcat **var/lib/tomcat7/webapps** 目录
    2. 在 **webapps** 目录上设置恰当的权限
    3. 在 **var/lib/tomcat7/conf/server.xml** 的 Tomcat 服务器配置文件中指定 `unpackWARS` 和 `autoDeploy` 的值
    4. 在 Tomcat 文件 **var/lib/tomcat7/conf/tomcat-users.xml** 中列出有效且可用的用户

    >**备注**： 服务启动时，Tomcat 将提取并部署 .war 文件。这就是在 **init.pp** 文件中重启服务的原因。完成本实验室后，可以打开并查看 Tomcat 文件 **server.xml** 和 **tomcat-users.xml** 的内容。

##### 第 5.6 部分：启动排序服务

>**备注**：PU MRP 应用调用排序服务，该服务是一种 REST API，用于管理 Mongo DB 中的顺序。排序服务编译为 .jar 文件。需要将. jar 文件复制到节点。然后，该节点在后台运行排序服务，以便它可以侦听 REST API 请求。

1.  在 Nano 编辑器界面中，将以下 `orderingservice` 类附加到 **init.pp** 文件的末尾，以确保排序服务运行：

    ```puppet
    class orderingservice {
      package { 'openjdk-7-jre':
        ensure => 'installed',
      }

      file { '/opt/mrp':
        ensure => 'directory'
      }->
      wget::fetch { 'orderingsvc':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/ordering-service-0.1.0.jar',
        destination => '/opt/mrp/ordering-service.jar',
        cache_dir => '/var/cache/wget',
        timeout => 0,
      }->
  
      exec { 'stoporderingservice':
        command => "pkill -f ordering-service",
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "pgrep -f ordering-service"
      }->

      exec { 'stoptomcat':
        command => 'service tomcat7 stop',
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "test -f /etc/init.d/tomcat7",
      }->
      exec { 'orderservice':
        command => 'java -jar /opt/mrp/ordering-service.jar &',
        path => '/usr/bin:/usr/sbin:/usr/lib/jvm/java-8-openjdk-amd64/bin',
      }->
      exec { 'wait':
        command => 'sleep 20',
        path => '/bin',
        notify => Tomcat::Service['default']
      }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，然后按 **Enter** 键以保存更改并使文件保持打开。 

    >**备注**： 我们来看看 orderingservice 类。

    第 1 行。创建名为 `orderingservice` 的 `class (resource)`。
    第 2-4 行。使用 Puppet 的包资源安装运行应用程序所需的 Java Runtime Environment (JRE)。
    第 6-8 行。验证目录 `/opt/mrp` 是否存在，并按需进行创建。
    第 9-14 行。使用 `wget` 检索排序服务二进制文件，并将其放在 `/opt/mrp` 中。
    第 12 行。指定一个缓存目录，以确保该文件仅下载一次。
    第 15-19 行。如果排序服务正在运行，请停止它。
    第 20-24 行。如果 tomcat7 服务正在运行，请停止它。
    第 25-28 行。启动排序服务。
    第 29-33 行。在通知 tomcat 服务之前等待 20 秒钟，以便有时间启动排序服务。这些操作将触发服务刷新。Puppet 将重新应用为服务定义的状态（即，如果服务尚未运行，则启动它）。

    >**备注**： 在运行 java 命令之后需要稍等片刻，因为需要先运行此服务，然后才能启动 Tomcat。否则，Tomcat 将占用排序服务侦听传入 API 请求所需的端口。

##### 第 5.7 部分：完成 mrpapp 资源

1.  在 Nano 编辑器界面中，向上滚动到 **init.pp** 文件的顶部，并通过删除与我们在实验室的前面部分测试过的 **dummy.txt** 文件相关的内容来删除 `mrpapp` 类定义。 
1.  在 Nano 编辑器界面中，添加更新后的 `mrpapp` 类定义：

    ```puppet
    class mrpapp {
      class { 'configuremongodb': }
      class { 'configurejava': }
      class { 'createuserandgroup': }
      class { 'configuretomcat': }
      class { 'deploywar': }
      class { 'orderingservice': }
    }
    ```

    >**备注**： 对 `mrpapp` 类的修改可指定 **init.pp** 文件中所需的类，以便运行资源。

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。

    >**备注**： 对 **init.pp** 文件进行所有必要的修改后，它应该与 [Microsoft 的 PU MRP GitHub 存储库](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp)上的 **init.pp** 文件相同。如果愿意，可以从 GitHub 复制 **init.pp** 文件的内容并将其粘贴到本地 **init.pp** 文件中。

##### 第 5.8 部分：配置 .war 文件提取权限

>**备注**： 在 **init.pp** 文件中定义的部分配置涉及将 PU MRP 应用的 .war 文件复制到目录 **var/lib/tomcat7/webapps** 中。Tomcat 自动从 webapps 目录中提取 .war 文件。但是，在我们的实验室中，我们将在 **war.pp** 文件中设置完整的读取、写入和访问权限，以确保文件提取成功。

1.  在 Puppet 主机的 PuTTY 会话中运行以下命令，以在 Nano 编辑器中打开 **war.pp** 文件：

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/tomcat/manifests/war.pp
    ```

1.  在 Nano 编辑器界面中，滚动至该文件底部，然后将现有的 `mode => '0640'` 条目替换为 `mode => '0777'`，以修改预期的权限。这将生成以下内容：

    ```puppet
        file { "tomcat::war ${name}":
          ensure    => file,
          path      => "${_deployment_path}/${_war_name}",
          owner     => $user,
          group     => $group,
          mode      => '0777',
          subscribe => Archive["tomcat::war ${name}"],
        }
      }
    }
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。

    >**备注**： 我们来看看 war.pp 文件的这一部分。

    第 1 行。指定文件名，该文件名包含在 Tomcat 模块 `tomcat::war` 中。
    第 2 行。在执行操作之前，请确保文件存在。
    第 3 行。定义 Web 目录的路径，我们之前将其设置为 `/var/lib/tomcat7/webapps`。像 **init.pp** 文件中指定的那样，会将 PU MRP 应用的 .war 文件复制到此位置。
    第 4-5 行。设置文件的用户和组所有者。
    第 6 行。将文件权限模式设置为 `0777`，这将向所有人授予读取、写入和执行权限。不应在生产环境中使用 `777` 权限。

#### 任务 6：在 Puppet 节点上运行 Puppet 配置

在此任务中，你将在 Puppet 节点上触发配置更新，并验证是否可以成功部署 PU MRP 应用。

1.  切换到 Puppet 节点的 PuTTY 会话，然后重新运行以下命令以测试新配置：

    ```bash
    sudo puppet agent --test
    ```

    >**备注**： 第一次运行可能需要花几分钟时间，因为必须下载并安装所有必需的组件。在后续运行期间，Puppet 代理只验证现有环境是否配置正确。 

    >**备注**： 忽略有关 `openjdk-7-jre` 的错误消息。这在意料之中。OpenJDK 6 和 7 在 Ubuntu 16.04 中不可用。

1.  若要验证 Tomcat 是否正常运行，请启动 Web 浏览器，然后导航到由以下部分构成的 URL：`http:\\` 前缀，后跟托管 Puppet 节点的 Azure VM 的 DNS 名称，后跟“:9080”。验证浏览器是否显示 Tomcat 登陆页面。
1.  要验证 PU MRP 应用是否正常运行，请在同一浏览器标签页中，将 `/mrp` 后缀附加到 URL。验证浏览器是否显示 PU MRP 应用的“欢迎”页。

### 练习 3：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**： 请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03a-RG')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03a-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**： 该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 回顾

在本实验室中，你学习了如何使用 Puppet 实验室中的 Puppet 将 PartsUnlimted MRP 应用程序（PU MRP 应用）部署到 Azure VM。 
