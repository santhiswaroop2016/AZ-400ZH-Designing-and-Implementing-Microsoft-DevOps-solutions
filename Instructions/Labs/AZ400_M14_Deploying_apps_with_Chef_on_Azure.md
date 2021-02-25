---
lab:
    title: '实验室：在 Azure 上使用 Chef 部署应用”
    module: '模块 14：通过 Azure 提供的第三方基础结构即代码工具”
---

# 实验室：在 Azure 上使用 Chef 部署应用
# 学生实验室手册

## 实验室概述

在本实验室中，我们将使用 Chef 服务器将 PartsUnlimted MRP 应用程序（PU MRP 应用）部署到 Azure VM。本实验室阐述了基于 Azure VM 的部署上下文中 Chef 的关键特性和功能。 

Chef 支持以下部署选项：

- 配置为 Chef 服务器的受支持 Linux 分发版。为此，你可以使用基于标准 Ubuntu 映像的 Azure VM。
- 预先配置的 Chef Automate 映像。可以基于 Chef Automate 映像部署 Azure VM。此映像包括 Chef 服务器。
- 托管的 Chef 服务器。可以在 https://manage.chef.io 上注册托管的 Chef 服务器帐户，然后使用托管的服务器配置 Chef 环境。

在本实验室中，你将基于 Chef Automate 映像部署 Azure VM，并利用 30 天的免费试用许可证。 

本实验室将包括以下概要步骤：

- 在 Azure 上部署 Chef Automate 服务器。
- 通过配置工作站来连接 Chef Automate 服务器并与之交互。
- 通过为 PU MRP 应用程序及其依赖项创建一个 cookbook 和 recipe，以使其自动进行安装。
- 将 cookbook 上传到 Chef Automate 服务器。
- 通过创建一个角色来定义一组可应用于多个服务器的基准 cookbook 和属性。
- 创建一个节点以将配置部署到 Linux VM。
- 通过启动节点 Linux VM 来添加角色，这会有效地依靠 Chef 来部署 PU MRP 应用程序。

## 实验室环境

实验室环境包括两个 Azure VM 和实验室计算机。Azure VM 包括：

- 托管 Chef 服务器的 Chef Automate
- 一个 Chef 节点，你将使用 Chef 在其中部署 PU MRP 应用程序。

你将使用 Azure 资源管理器模板来部署两个 Azure VM。实验室计算机将充当 Chef 工作站，你将使用该工作站来连接和管理 Chef 服务器。Chef 工作站可以运行 Windows Server 或客户端操作系统、Linux 或 MacOS 的任何当前版本。 

## 目标

完成本实验室后，你将能够：

- 使用 Azure 资源管理器模板部署 Chef Automate 服务器和客户端
- 配置 Chef 工作站
- 创建一个 Chef cookbook 和 recipe，并将其上传到 Chef Automate 服务器
- 创建一个 Chef 角色并将其部署到客户端

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

### 练习 1：使用 Chef 服务器将 PartsUnlimted MRP 应用程序部署到 Azure VM 

在本练习中，你将使用 Chef 服务器将 PartsUnlimted MRP 应用程序部署到 Azure VM。

#### 任务 1：将 Chef Automate 服务器部署到 Azure VM

在此任务中，你将使用 Azure 资源管理器模板部署和配置 Azure VM。Azure VM 将充当 Chef Automate 服务器。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1.  打开另一个浏览器标签页，然后导航到 [Azure 资源管理器模板链接](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploychef.json)。这会自动重定向到 Azure 门户中的 **“自定义部署”** 边栏选项卡。
1.  在 Azure 门户的 **“自定义部署”** 边栏选项卡上，指定以下设置（其他设置保留默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 **az400m14l02a-RG** |
    | 区域 | 可以在其中部署 Azure VM 的 Azure 区域的名称 |
    | 管理员用户名 | **azureuser** |
    | 管理员密码 | **Pa55w.rd1234** |
    | 身份验证类型 | **密码** |
    | 诊断存储帐户名称 | 由 3 到 24 个字母和数字组成的任意唯一字符串，以字母开头 |
    | 新的或现有的诊断存储帐户 | **新** |
    | 位置 | 选择在其中部署 Azure VM 的同一 Azure 区域的名称（不带空格） |
    | 公共 IP 地址名称 | **az400m14l02b-pip** |
    | 公共 IP DNS 名称 | 由 3 到 63 个字母和数字组成的任意唯一字符串，以字母开头 |
    | 存储帐户名称 | 与你为诊断存储帐户选择的名称相同 |
    | 虚拟网络名称 | **az400m14l02b-vnet** |
    | VM 名称 | **az400m14chs-vm** |

1.  在 **“自定义部署”** 边栏选项卡上，单击 **“查看 + 创建”**，确认验证过程完成，然后单击 **“创建”**。

    >**备注**： 等待部署完成。该操作需约 10 分钟。

1.  部署完成后，在 Azure 门户中，使用页面顶部的 **“搜索资源、服务和文档”** 文本框搜索并选择 **“虚拟机”** 资源，然后在 **“虚拟机”** 边栏选项卡上，选择代表新部署的 **az400m14chs-vm** 虚拟机的条目。
1.  在 **“az400m14chs-vm”** 边栏选项卡上，将鼠标指针悬停在 **“DNS 名称”** 条目上，然后将其复制到剪贴板。 
1.  打开一个新的浏览器标签页，然后导航到刚复制到剪贴板中的 DNS 名称。使用 **https** 前缀。 

    >**备注**： 忽略任何证书错误。这在预料之中。 

1.  确认你可以访问 Chef Automate 登录提示。

    >**备注**： 你需要先完成下一个任务，然后再登录 Chef Automate。 

#### 任务 2：配置 Chef 工作站

在此任务中，你将通过安装 Chef 初学者工具包将实验室计算机配置为 Chef 工作站。你将通过 Chef 工作站连接和管理 Chef 服务器。

1.  在实验室计算机上，启动另一个 Web 浏览器窗口，导航到 [Chef 开发工具包（Chef DK）](https://downloads.chef.io/chefdk)下载页面，选择与你的实验室虚拟机 (Windows 10) 匹配的 Chef 开发工具包版本，单击 **“下载”**，在 **“下载 ChefDK”** 弹出窗口中，提供所需的联系人信息，然后再次单击 **“下载”**。

    >**备注**： 截至 2020 年 12 月，Chef 开发工具包的最新版本为 4.12.0。适用于 Windows 10 的 Chef 开发工具包 4.12.0-1-x64 已在本实验室中进行了测试。

1.  下载 Chef 开发工具包的安装程序后，请使用默认设置完成安装，然后双击出现的桌面快捷方式以启动 **“管理员: ChefDK”** PowerShell 窗口。 
1.  在 **“管理员: ChefDK”** PowerShell 窗口中，运行以下命令以验证安装：

    ```
    chef verify
    ```

1.  当系统提示你接受产品许可证时，键入 **“是”**，然后按 **Enter** 键。
1.  如果验证失败，请在 **“管理员: ChefDK”** PowerShell 窗口中，运行以下命令以使用名称和电子邮件地址配置全局 Git 变量，然后再次重新运行 `chef verify`： 

    ```
    git config --global user.name "azureuser"
    git config --global user.email "azureuser@partsunlimitedmrp.local"
    ```

    >**备注**： 你将使用 Git 来存储 Chef 配置详细信息。

    >**备注**： 使 PowerShell 窗口保持打开状态，你将在本实验室后面部分再次使用它。

    >**备注**： 要连接和管理 Chef Automate，你需要标识托管其安装的 Azure VM 的 vmId 属性。vmId 是分配给 Azure 中每个 VM 的唯一值。可以通过多种方式获得 vmId，包括 Azure PowerShell 和 Azure CLI。在本实验室中，你将为此使用 PuTTY。

1.  在实验室计算机上的 Web 浏览器窗口中，导航到 [PuTTY 下载页面](https://putty.org/)，下载 PuTTY 安装程序，然后使用默认设置运行安装。
1.  安装完成后，在 **“开始”** 菜单中，展开 **“PuTTY(64 位)”** 文件夹，然后单击 **“PuTTY”** 图标以打开 **“PuTTY 配置”** 窗口。
1.  在 **“PuTTY 配置”** 窗口的 **“主机名(或 IP 地址)”** 文本框中，键入在上一个任务结束时标识的 DNS 名称，然后单击 **“打开”**。
1.  当系统出现提示时，在 **“PuTTY 安全警报”** 窗口中，单击 **“是”**。
1.  当系统出现提示时，在 **PuTTY** 控制台窗口中，使用用户名 **azureuser** 和密码 **Pa55w.rd1234** 登录。
1.  登录后，在 PuTTY 控制台窗口中，运行以下命令以标识 vmId：

    ```bash
    sudo chef-marketplace-ctl show-instance-id
    ```

    >**备注**： 请记录 vmId 的值。你将在此任务的稍后部分使用它。

1.  在 PuTTY 控制台窗口中，运行以下命令以终止会话：

    ```bash
    exit
    ```

    >**备注**： 你需要使用 Chef 初学者工具包来访问在上一个任务中安装的 Azure VM 中运行的 Chef Automate。要访问 Chef 初学者工具包，请将 /biscotti/setup 追加到在上一个任务结束时标识的 DNS 名称。获得的结果是类似于以下格式的 URL（其中的 `<DNS_name>` 占位符表示在上一个任务结束时标识的 DNS 名称）：

    ```
    https://<DNS_name>/biscotti/setup
    ```

1.  在 Web 浏览器窗口中，导航到上一步中构建的 URL，忽略任何与证书相关的错误，然后进入 **“设置授权”** 页面。 
1.  在 **“设置授权”** 页面上，当系统提示你提供 **vmId** 时，键入你在 **PuTTY** 会话中标识的值，然后单击 **“授权”**。
1.  当系统提示你提供 Chef Automate 设置的详细信息时，请指定以下设置，然后单击 **“设置 Chef Automate”**：

    | 设置 | 值 |
    | --- | --- |
    | 输入你的名字 | **Azure** |
    | 输入你的姓氏 | **User** |
    | 输入你的用户名 | **azureuser** |
    | 输入你的电子邮件地址 | **azureuser@partsunlimitedmrp.local** |
    | 输入密码 | **Pa55w.rd1234** |
    | 创建一个支持帐户以启用 Chef Automate 订阅中包含的 Chef Base 支持 | **关闭** |
    | 我已阅读并接受 EULA 和主协议 | **开启** |

    >**备注**：这将应用**默认**组织名称，因为我们没有在 Chef Automate 配置设置中指定组织名称。可以选择在 Chef Automate 服务器中指定组织名称。本实验室后面部分的 **knife.rb** 文件中将包含该组织值。 

1.  当系统出现提示时，单击 **“下载初学者工具包”**。这将触发下载名为 starter-kit.zip 的文件，其中包含可帮助设置 Chef 的预配置文件。 

    >**备注**： Chef 初学者工具包包含用户凭据和证书详细信息。初始注册期间会生成不同的凭据和详细信息，这些凭据和详细信息特定于你提供的用户和组织的详细信息。确保不要在多个用户或注册中重复使用 Chef 初学者工具包文件。

    >**备注**： 下载 Chef 初学者工具包后， **“登录到 Chef Automate”** 按钮将变为可用。 

1.  在 Web 浏览器窗口中，单击 **“登录到 Chef Automate”** 按钮，然后使用你在 **“Chef Automate 设置”** 网页上提供的用户名 **azureuser** 和密码 **Pa55w.rd1234** 登录到 Chef Automate。此时将显示 Chef Automate 仪表板。
1.  在实验室计算机上，打开 “文件资源管理器”，然后将 **“Chef 初学者工具包”** zip 存档文件的内容从 **“Downloads”** 文件夹中提取到 **“C:\\Labfiles\\chef”** 文件夹。
1.  在“文件资源管理器”中，导航到 **“C:\\Labfiles\\chef\\chef-repo\\.chef”** 文件夹，然后在记事本中打开 **knife.rb** 文件。 
1.  在 **knife.rb** 文件中，验证 **chef_server_url** 名称中包含的完全限定的域名与你在上一个任务中标识的 Azure VM DNS 名称匹配，并且组织名称已设置为 **“默认”**，然后关闭“记事本”窗口。
1.  切换回 **“管理员: Chef DK”** PowerShell 窗口，并运行以下命令以启动新存储库：

    ```powershell
    Set-Location -Path 'C:\Labfiles\chef\chef-repo'
    git init
    git add -A
    git commit -m "starter kit commit"
    ```

    >**备注**： Chef 服务器具有不受信任的 SSL 证书。因此，必须手动信任 SSL 证书，以便 Chef 工作站可以与 Chef 服务器通信。要进行此更改，将通过运行以下命令为 Chef 导入有效的 SSL 证书：

    ```chef
    knife ssl fetch
    ```

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以列出 chef-repo 的当前内容：

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令，以将 Chef 工作站存储库的内容与 Chef Automate 基于 Azure VM 的存储库的内容同步。 

    ```chef
    knife download /
    ```

    >**备注**： 此命令从 Chef Automate 服务器下载整个 Chef 存储库。

    >**备注**： 可能会出现与 ACL 相关的错误，具体取决于用户帐户详细信息，但在本实验室中，你可以忽略这些错误。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，在运行 `knife download /` 命令后再次运行以下命令以列出 chef-repo 的当前内容，并注意在 chef-repo 目录中创建的其他文件和文件夹：

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以将新添加的文件提交到 Git 存储库中：

    ```git
    git add -A
    git commit -m "knife download commit"
    ```

#### 任务 3：创建一个 Chef cookbook 和 recipe，并将其上传到 Chef Automate 服务器

在此任务中，你将为 PU MRP 应用的依赖项创建 cookbook 和 recipe，以自动安装 PU MRP 应用程序，并将这些 cookbook 和 recipe 上传到 Chef 服务器。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以使用 Chef knife 工具在 **chef-repo** 的 **cookbooks** 子目录中生成一个 cookbook 模板：

    ```chef
    Set-Location -Path '.\cookbooks'
    chef generate cookbook mrpapp
    ```

    >**备注**：cookbook 是用于配置应用程序或功能的一组任务。它定义某一方案以及支持该方案所需的所有内容。在 cookbook 中，有一系列定义 cookbook 要执行的一组操作的 recipe。cookbook 和 recipe 是采用 Ruby 语言编写的。运行的 chef generate cookbook 命令在 **“chef-repo\\cookbooks”** 目录中创建了一个 **mrpapp** 目录。mrpapp 目录包含定义 cookbook 和默认 recipe 的所有样本代码。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以打开文件 **metadata.rb** 进行编辑：

    ```powershell
    notepad .\mrpapp\metadata.rb
    ```

    >**备注**： Cookbook 和 recipe 可以利用其它 cookbook 和 recipe。cookbook 将使用已有的 recipe 来管理 APT 存储库。 

1.  在显示 **metadata.rb** 文件内容的记事本窗口中，在文件末尾添加以下行，然后保存并关闭记事本窗口：

    ```chef
    depends 'apt'
    ```

    >**备注**： 接下来，我们需要为 recipe 安装三个依赖项： **apt** cookbook、**Windows** cookbook 和 **chef-client** cookbook。可以从[官方 Chef cookbook 存储库](https://supermarket.chef.io/cookbooks)中下载这三个 cookbook，并使用 `knife cookbook site` 命令进行安装。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以使用 `knife cookbook site` 命令下载并安装 cookbook：

    ```chef
    knife cookbook site install apt
    knife cookbook site install windows
    knife cookbook site install chef-client
    ```

    >**备注**： 接下来，你需要从 [Microsoft Parts Unlimited MRP GitHub 存储库](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb)中复制 **default.rb** recipe 的全部内容。

1.  在实验室计算机上，启动另一个 Web 浏览器窗口，导航到以下链接，以 RAW 格式显示 default.rb recipe，并将网页内容复制到剪贴板： **https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb**
1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以在记事本中打开文件 **C:\\Labfiles\\chef\\chef-repo\\cookbooks\\mrpapp\\recipes\\default.rb**：

    ```powershell
    notepad .\mrpapp\recipes\default.rb
    ```

1.  在“记事本”窗口中，查看文件的内容并验证其是否如下所示：

    ```chef
    #
    # Cookbook:: mrpapp
    # Recipe:: 默认
    #
    # 版权:: 2020，作者，保留所有权利。
    ```

1.  在“记事本”窗口中，将剪贴板的内容附加到文件末尾，从新行开始，保存所做的更改，然后关闭“记事本”窗口。

    >**备注**： 以下内容说明了复制的 recipe 将如何预配应用程序，并逐行说明了 **default.rb** 内容的含义：

    >**备注**： recipe 将运行 `apt` 资源。这将导致 recipe 在运行之前会执行 `apt-get update` 命令。此命令可确保本地包的版本是最新版本：

    ```chef
     	# 运行 apt-get 更新
     	include_recipe "apt"
    ```

    >**备注**： 接下来，添加一个 `apt_repository` 资源，以确保 **OpenJDK** 存储库是 apt 存储库列表的一部分，并且它是最新的：

    ```chef
 	# 添加 Open JDK apt 存储库
 	apt_repository 'openJDK' do
 		uri 'ppa:openjdk-r/ppa'
 		distribution 'trusty'
 	end
    ```

    >**备注**： 接下来，使用 `apt-package` recipe 来确保已安装 **OpenJDK** 和 **OpenJRE**。使用无头版本，因为完整版本依赖不建议使用的旧版包：

    ```chef
 	# 安装 JDK 和 JRE
 	apt_package 'openjdk-8-jdk-headless' do
 		action :install
 	end

 	apt_package 'openjdk-8-jre-headless' do
 		action :install
     	end
    ```

    >**备注**： 接下来，将环境变量 `JAVA_HOME` 和 `PATH` 设置为引用 OpenJDK：

    ```chef
 	# 设置 Java 环境变量
 	ENV['JAVA_HOME'] = "/usr/lib/jvm/java-8-openjdk-amd64"
     	ENV['PATH'] = "#{ENV['PATH']}:/usr/lib/jvm/java-8-openjdk-amd64/bin"
    ```

    >**备注**： 接下来，安装 MongoDB 数据库引擎和 Tomcat Web 服务器：

    ```chef
 	# 安装 MongoDB
 	apt_package 'mongodb' do
 		action :install
 	end

 	# 安装 Tomcat 7
 	apt_package 'tomcat7' do
 		action :install
     	end
    ```

    >**备注**： 至此，所有依赖项均已安装。现在，可以开始配置应用程序了。首先，需要确保 MongoDB 数据库中包含一些基线数据。`remote_file` 资源将文件下载到指定位置。此任务是幂等的，在此上下文中，这意味着如果服务器上的文件与本地文件具有相同的校验和，则不会执行任何操作。我们也使用 `notifies` 命令。这样，如果资源运行时存在文件的新版本，则会向指定资源发送一条通知，通知它运行新文件。

    ```chef
 	# 加载 MongoDB 数据
 	remote_file 'mongodb_data' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/deploy/MongoRecords.js'
 		path './MongoRecords.js'
 		action :create
 		notifies :run, "script[mongodb_import]", :immediately
     	end
    ```

    >**备注**： 接下来，使用 `script` 资源来设置命令行脚本，通过运行该脚本，可加载在上一步中下载的 MongoDB 数据。`script` 资源将其 `action` 参数设置为 `nothing`，这意味着该脚本仅在收到运行通知时才运行。该资源仅在收到上一步中指定的 `remote_file` 资源的通知时才运行。因此，每次上传新版本的 **MongoRecord.js** 文件时，recipe 都会下载并导入该文件。如果 **MongoRecords.js** 文件没有更改，则不会下载或导入任何内容！

    ```chef
 	script 'mongodb_import' do
 		interpreter "bash"
 		action :nothing
 		code "mongo ordering MongoRecords.js"
     	end
    ```

    >**备注**： 接下来，设置 Tomcat 用于运行 PU MRP 应用程序的端口。它使用 `script` 资源调用正则表达式来更新 **etc/tomcat7/server.xml** 文件。`not_if` 操作是一个 guard 语句。如果 `not_if` 操作中的代码返回 `true`，则该资源将不会执行。这样可以确保脚本仅按需运行。

    >**备注**： 我们将引用一个名为 `#{node[’tomcat’][’mrp_port’]}`的属性。我们尚未定义此值，但是将在下一个任务中进行定义。通过属性，可以设置变量，这样就可以在一台服务器上的一个端口上以及另一台服务器上的另一个端口上部署 PU MRP 应用程序。如果端口更改，使用 `notifies` 来调用服务重启。

    ```chef
 	# 设置 tomcat 端口
 	script 'tomcat_port' do
 		interpreter "bash"
 		code "sed -i 's/Connector port=\".*\" protocol=\"HTTP\\/1.1\"$/Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP\\/1.1\"/g' /etc/tomcat7/server.xml"
 		not_if "grep 'Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP/1.1\"$' /etc/tomcat7/server.xml"
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    >**备注**： 现在，可以下载 PU MRP 应用程序，并开始在 Tomcat 中运行它。如果获得新版本，则向 Tomcat 服务发出重启信号：

    ```chef
 	# 安装 PU MRP 应用，并在必要时重启 Tomcat 服务
 	remote_file 'mrp_app' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/mrp.war'
 		action :create
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    >**备注**： 可以定义 Tomcat 服务的预期状态，在此特定情况下，这意味着该服务应该正在运行。这将导致脚本检查 Tomcat 服务的状态，并在必要时启动它。

    ```chef
 	# 确保 Tomcat 正在运行
 	service 'tomcat7' do
 		action :start
 	end
    ```

    >**备注**： 最后，可以确保 `ordering_service` 正在运行。这结合使用了 `remote_file` 和 `script` 资源，以检查是否需要终止并重启 ordering_service。 

    ```chef
 	remote_file 'ordering_service' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/ordering-service-0.1.0.jar'
 		path './ordering-service-0.1.0.jar'
 		action :create
 		notifies :run, "script[stop_ordering_service]", :immediately
 	end

 	# 终止排序服务
 	script 'stop_ordering_service' do
 		interpreter "bash"
 	# 仅在收到通知时运行
 		action :nothing
 		code "pkill -f ordering-service"
 		only_if "pgrep -f ordering-service"
 	end

 	# 启动排序服务。
 	script 'start_ordering_service' do
 		interpreter "bash"
 		code "/usr/lib/jvm/java-8-openjdk-amd64/bin/java -jar ordering-service-0.1.0.jar &"
 		not_if "pgrep -f ordering-service"
 	end
    ```

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以提交添加到 Git 存储库的文件：

    ```git
    git add .
    git commit -m "mrp cookbook commit"
    ```

    >**备注**： 现在我们已经创建了 recipe 并安装了依赖项，可以将 cookbook 和 recipe 上传到 Chef Automate 服务器了。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令，以使用 `knife cookbook upload` 命令将 cookbook 和 recipe 上传到 Chef Automate 服务器：

    ```chef
    knife cookbook upload mrpapp --include-dependencies
    knife cookbook upload chef-client --include-dependencies
    ```

#### 任务 4：创建 Chef 角色

在本练习中，你将使用 `knife` 命令创建一个角色。该角色将定义一组可应用于多个服务器的基线 cookbook 和属性。

>**备注**： 有关详细信息，请参阅[“Chef 文档”页面的 Knife 角色](https://docs.chef.io/knife_role.html)。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以在记事本中打开 **knife.rb** 文件：

    ```powershell
    notepad C:\Labfiles\chef\chef-repo\.chef\knife.rb
    ```

1.  在“记事本”窗口中，将以下内容附加到文件末尾（从新的一行开始），保存所做的更改，然后关闭“记事本”窗口：

    ```chef
    knife[:editor] = "notepad"
    ```

    >**备注**： 添加此行会指定记事本作为编辑器，以便在我们创建角色时打开 knife.rb 文件，这将在下一步中进行。如果未在 knife.rb 中指定编辑器，则会收到一条错误消息，指示你设置编辑器环境变量。

1.  在 **“管理员: Chef DK”** PowerShell 窗口中，运行以下命令以创建角色，我们将该角色命名为 **“partsrole”**。

    ```chef
    knife role create partsrole
    ```

    >**备注**： 此命令将打开“记事本”窗口，其中显示表示角色定义结构的 JSON 内容：

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {

      },
      "override_attributes": {

      },
      "chef_type": "role",
      "run_list": [

      ],
      "env_run_lists": {

      }
    }
    ```

1.  在记事本窗口中，将 `default_attributes` 部分更改为：

    ```json
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
    ```

1. 将 override_attributes 更新为：

    ```json
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
    ```

1. 将 run_list 更新为：

    ```json
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
    ```

1.  验证完成的文件是否具有以下内容，保存所做的更改，然后关闭“记事本”窗口：

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
      "chef_type": "role",
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
      "env_run_lists": {

      }
    }
    ```

1.  切换回 **“管理员: Chef DK”** PowerShell 窗口，并验证 `knife role create partsrole` 已成功完成。 

    >**备注**： 请确保关闭“记事本”窗口。通过 `Created role[partsrole]` 消息指示成功完成。 

#### 任务 5：启动 PU MRP 应用服务器并部署 PU MRP 应用程序

在本练习中，你将使用 **knife** 启动 PU MRP 应用程序服务器，并为其分配 PU MRP 应用程序角色。

>**备注**： 你将首先预配 Linux VM，然后将其配置为 Chef 客户端，并使用在上一个任务中创建的角色将 PU MRP 应用程序部署到其中。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [Azure 资源管理器模板链接](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploylinux.json)。这会自动重定向到 Azure 门户中的 **“自定义部署”** 边栏选项卡。
1.  在 Azure 门户中，在 **“自定义部署”** 边栏选项卡上，单击 **“编辑模板”**
1.  在 **“编辑模板”** 边栏选项卡上的模板概述窗格中，展开 **“变量”** 部分，然后单击 **“vmSize”**。
1.  在模板详细信息部分中，将 `"vmSize": "Standard_A1",` 更改为 `"vmSize": "Standard_D2s_v3",` 并单击 **“保存”**。
1.  返回 **“自定义部署”** 边栏选项卡，指定以下设置（其他设置保留默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 **az400m14l02b-RG** |
    | 区域 | 在本实验室前面部分中用于部署 Chef Automate Azure VM 的 Azure 区域的名称 |
    | 管理员用户名 | **azureuser** |
    | 管理员密码 | **Pa55w.rd1234** |
    | DNS 标签前缀 | 由 3 到 63 个字母和数字组成的任意唯一字符串，以字母开头 |
    | Ubuntu OS 版本 | **16.04.0-LTS** |

1.  在 **“自定义部署”** 边栏选项卡上，单击 **“查看 + 创建”**，确认验证过程完成，然后单击 **“创建”**。

    >**备注**： 等待部署完成。该过程需要约 2 分钟。

1.  部署完成后，在 Azure 门户中，使用页面顶部的 **“搜索资源、服务和文档”** 文本框搜索并选择 **“虚拟机”** 资源，然后在 **“虚拟机”** 边栏选项卡上，选择代表新部署的虚拟机 **mrpUbuntuVM** 的条目。
1.  在 **“mrpUbuntuVM”** 边栏选项卡上，将鼠标指针悬停在 **DNS 名称**条目上，然后将其复制到剪贴板。 
1.  切换回 **“管理员: Chef DK”** PowerShell 窗口，运行以下命令以使用 **knife** 工具来启动新部署的 Azure VM（其中的 `<DNS_name>` 占位符表示在上一个任务结束时标识的 DNS 名称）：

    ```chef
    knife bootstrap <DNS_name> --ssh-user azureuser --ssh-password Pa55w.rd1234 --node-name mrp-app --run-list role[partsrole] --sudo --verbose
    ```
1.  当系统提示你是否确定要继续连接时，键入 **Y** 并按 **Enter** 键。

    >**备注**： 请等待加入脚本完成。该操作需约 3 分钟。该脚本执行以下步骤：

    - 安装 Chef 客户端组件。
    - 分配 `mrp` Chef 角色。
    - 运行 `mrpapp` recipe。

1.  脚本完成后，切换到显示 Azure 门户的浏览器窗口，打开另一个浏览器标签页，然后导航到由前缀 **http://**、`mrpUbuntuVM` Azure VM 的 DNS 名称和后缀 `:9080/mrp/` 组成的 URL（该 URL 的最终格式为 `http://<DNS_name>:9080/mrp/`，其中的 `<DNS_name>` 占位符表示在上一个任务结束时标识的 DNS 名称）。
1.  验证浏览器标签页是否显示 PU MRP 应用程序的登陆页面。
1.  切换到显示 Chef Automate 服务器仪表板的浏览器窗口，在仪表板页面顶部，单击 **“节点”**，并验证它是否包含标记为 **mrp-app** 的单个节点。

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**： 请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**： 该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 回顾

在本实验室中，你学习了如何使用 Chef 服务器将 PartsUnlimted MRP 应用程序（PU MRP 应用）部署到 Azure VM。 
