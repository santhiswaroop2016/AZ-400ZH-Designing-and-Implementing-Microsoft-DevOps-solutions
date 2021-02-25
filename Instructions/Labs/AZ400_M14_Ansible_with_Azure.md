---
lab:
    title: '实验室：Ansible 与 Azure'
    module: '模块 14：通过 Azure 提供的第三方基础结构即代码工具'
---

# 实验室：Ansible 与 Azure
# 学生实验室手册

## 实验室概述

在本实验室中，我们将使用 Ansible 部署、配置和管理 Azure 资源。 

Ansible 是声明性配置管理软件。它依赖于托管计算机的预期配置描述，这种描述通常采用 playbook 形式。Ansible 自动应用该配置并维护它持续运行，从而解决任何潜在差异。Playbook 使用 YAML 进行格式设置。

与大多数其他配置管理工具（如 Puppet 或 Chef）不同，Ansible 是无代理的，这意味着它不需要在托管计算机中安装任何软件。Ansible 使用 SSH 来管理 Linux 服务器，使用 Powershell Remoting 来管理 Windows 服务器和客户端。

为了与资源而非操作系统（例如可通过 Azure 资源管理器访问的 Azure 资源）进行交互，Ansible 支持称为模块的扩展。因此可使用 Python 高效编写 Ansible，并将模块作为 Python 库实现。Ansible 依赖于 [GitHub 托管模块](https://github.com/ansible-collections/azure)来管理 Azure 资源。

Ansible 要求将托管资源记录在主机清单中。对于某些系统（包括 Azure），Ansible 支持动态清单，以便在运行时动态生成主机清单。

本实验室将包括以下概要步骤：

- 使用 Azure Cloud Shell 部署两个 Azure VM
- 生成动态清单
- 使用 Azure CLI 在 Azure 中创建 Ansible 控制 VM
- 在 Azure VM 上安装和配置 Ansible
- 下载 Ansible 配置和示例 playbook 文件
- 在 Azure AD 中创建和配置服务主体
- 配置 Azure AD 凭据和 SSH 以用于 Ansible
- 使用 Ansible playbook 部署 Azure VM
- 使用 Ansible playbook 配置 Azure VM
- 使用 Ansible playbook 在 Azure VM 上进行配置和预期状态管理
- 结合使用 Ansible 和 Azure 资源管理器模板，促进 Azure 资源的配置和预期状态管理

## 目标

完成本实验室后，你将能够：

- 使用 Azure Cloud Shell 部署 Azure VM
- 生成 Ansible 动态清单
- 使用 Azure CLI 在 Azure 中创建 Ansible 控制 VM
- 在 Azure VM 上安装和配置 Ansible
- 下载 Ansible 配置和示例 playbook 文件
- 在 Azure AD 中创建和配置服务主体
- 配置 Azure AD 凭据和 SSH 以用于 Ansible
- 使用 Ansible playbook 部署 Azure VM
- 使用 Ansible playbook 配置 Azure VM
- 使用 Ansible playbook 在 Azure VM 上进行配置和预期状态管理
- 结合使用 Ansible 和 Azure 资源管理器模板，促进 Azure 资源的配置和预期状态管理

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

### 练习 1：使用 Ansible 部署、配置和管理 Azure VM

在本练习中，你将使用 Ansible 部署、配置和管理 Azure VM。

#### 任务 1：使用 Azure Cloud Shell 创建两个 Azure VM

在此任务中，你将使用 Azure Cloud Shell 创建两个 Azure VM。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的 **“Cloud Shell”** 图标。 

    >**备注**：或者，可以直接通过导航到 [https://shell.azure.com](https://shell.azure.com) 来访问 Cloud Shell。

1.  如果提示选择 **“Bash”** 或 **“PowerShell”**，请选择 **“Bash”**。 

    >**备注**： 如果这是你第一次打开 **“Cloud Shell”**，会看到 **“未装载任何存储”** 消息，请选择你在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建托管 Azure VM 的资源组（将 `<Azure_region>` 占位符替换为你打算在本实验室中将资源部署到的 Azure 区域的名称）：

    ```bash
    LOCATION=<Azure_region>
    RGNAME=az400m14l03arg
    az group create --name $RGNAME --location $LOCATION
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以将运行 Ubuntu 的第一个 Azure VM 部署到在上一步中创建的资源组：

    ```bash
    VM1NAME=az400m1403vm1
    az vm create --resource-group $RGNAME --name $VM1NAME --image UbuntuLTS --generate-ssh-keys --no-wait
    ```

    >**备注**： 无需等待部署完成，可直接继续执行下一个步骤。 

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以将运行 Ubuntu 的第二个 Azure VM 部署到同一个资源组：

    ```bash
    VM2NAME=az400m1403vm2
    az vm create --resource-group $RGNAME --name $VM2NAME --image UbuntuLTS --generate-ssh-keys
    ```

    >**备注**： 请等待部署完成，再继续下一步。该操作需要约 2 分钟。

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以使用标记 Nginx 来标记第一个 Azure VM，以便将其标识为 Nginx Web 服务器

    ```bash
    VM1ID=$(az vm show --resource-group $RGNAME --name $VM1NAME --query id --output tsv)
    az resource tag --tags Ansible=nginx --id $VM1ID
    ```

#### 任务 2：生成动态清单

在此任务中，你将生成一个动态 Ansible 清单，该清单面向在上一个任务中部署的两个 Azure VM。

>**备注**： 从 Ansible 2.8 开始，Ansible 已提供 Azure 动态清单插件。

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建名为 **“myazure_rm.yml”** 的新文件，并在 Nano 文本编辑器中将其打开：

    ```bash
    nano ./myazure_rm.yml
    ```

1.  在 Nano 编辑器界面中，粘贴以下内容：

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: auto

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。
1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以安装 Ansible Azure 模块：

    ```bash
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

    >**备注**： 从 Ansible 2.10 版开始，Azure 模块已与核心模块分开维护。要验证 Ansible 版本，请运行 `ansible --version`。

1.  在 Cloud Shell 窗格的 Bash 会话中，键入 `exit` 并按 **Enter** 键以退出会话。 

    >**备注**： 要使模块的安装生效，必须执行此操作。 

1.  在 Azure 门户的工具栏中，单击 Cloud Shell 图标以重启 **Cloud Shell 会话**。 
1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以执行 ping 测试并生成在 Azure 中运行的所有虚拟机（这些虚拟机位于资源组中，其名称已包含在 **myazure_rm.yml** 文件中）的动态清单：

    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

    >**备注**： 第一次运行该命令时，你需要确认目标 VM 的真实性，方法是键入 **“是”**，然后按 **Enter** 键。你可能需要为每个目标 Azure VM 执行此操作。验证真实性之后，应无需确认即可成功返回该命令的后续运行。如果需要，请多次运行上述命令，以生成成功的输出。

    >**备注**： 请忽略弃用警告。 

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以列出清单中的主机：

    ```bash
    ansible all -i ./myazure_rm.yml --list-hosts
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以列出清单中基于标记分组的主机：

    ```bash
    ansible-inventory -i ./myazure_rm.yml --graph
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以测试与清单中具有特定标记值的所有主机的连接性：

    ```bash
    ansible -i ./myazure_rm.yml -m ping tag_Ansible_nginx
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以列出第一个主机的基于 JSON 的属性表示形式（将 `<Ansible_host_name>` 替换为由 `ansible all -i ./myazure_rm.yml --list-hosts` 命令返回的第一个主机的名称）：

    ```bash
    ansible-inventory -i ./myazure_rm.yml --host <Ansible_host_name> | jq
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以列出清单中所有主机的基于 JSON 的属性表示形式：

    ```bash
    ansible-inventory -i ./myazure_rm.yml --list | jq
    ```

1.  关闭 Cloud Shell 窗格。

#### 任务 3：预配 Azure VM 作为 Ansible 控制计算机

在此任务中，你将使用 Azure CLI 部署一个 Azure VM，并对其进行配置以管理 Ansible 环境。

>**备注**： Ansible 静态主机清单使用 **etc/ansible/hosts 文件**。由于 Azure Cloud Shell 不提供根目录访问权限，存储选项限制为用户的 **$Home** 目录，因此为了通过动态清单实现 Ansible 管理，我们将部署一个运行 Linux 的 Azure VM 并将其配置为 Ansible 管理系统。

1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的 **“Cloud Shell”** 图标。 

    >**备注**： 或者，可以直接通过导航到 [https://shell.azure.com](https://shell.azure.com) 来访问 Cloud Shell。

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建一个将托管新的 Azure VM 的资源组：

    ```bash
    RG1NAME=az400m14l03arg
    VM1NAME=az400m1403vm1
    LOCATION=$(az vm show --resource-group $RG1NAME --name $VM1NAME --query location --output tsv)
    RG2NAME=az400m14l03brg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建一个将托管 Azure VM 的虚拟网络，该 VM 将用作 Ansible 管理系统：

    ```bash
    VNETNAME=az400m1403-vnet
    SUBNETNAME=ansible-subnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefix 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建一个将分配给 Azure VM（将用作 Ansible 管理系统）的网络适配器的公共 IP 地址：

    ```bash
    PIPNAME=az400m1403-pip
    az network public-ip create \
    --name $PIPNAME \
    --resource-group $RG2NAME \
    --location $LOCATION
    ```
 
1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以创建将用作 Ansible 控制计算机的 Azure VM：

    ```bash
    VM3NAME=az400m1403vm3
    az vm create \
    --name $VM3NAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --image UbuntuLTS \
    --vnet-name $VNETNAME \
    --subnet $SUBNETNAME \
    --public-ip-address $PIPNAME \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    >**备注**： 请等待部署完成，再继续下一步。该操作需要约 2 分钟。

    >**备注**： 预配完成后，在基于 JSON 的输出中，标识该输出中包含的 **“publicIpAddress”** 属性的值。 

1.  在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以使用 SSH 连接到新部署的 Azure VM：

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM3NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  出现确认继续操作的提示时，键入 **“是”** 并按 **Enter** 键，然后在提示提供密码时，键入 **Pa55w.rd1234**。

#### 任务 4：在 Azure VM 上安装和配置 Ansible

在此任务中，你将在上一个任务中部署的 Azure VM 上安装和配置 Ansible。

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令，以更新高级打包工具 (apt) 包列表，在其中包含最新版本和包详细信息：

    ```bash
    sudo apt update
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以安装 Ansible 所需的先决条件：

    ```bash
    sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以安装 Ansible 和所需的 Azure 模块：

    ```bash
    sudo apt-get install python3-pip
    sudo apt install ansible
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令，以安装 dnspython 包，允许 Ansible playbook 在部署前先验证 DNS 名称：

    ```bash
    sudo pip3 install dnspython 
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以安装 **jq** JSON 分析工具：

    ```bash
    sudo apt install jq
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以安装 Azure CLI：

    ```sh
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### 任务 5：下载 Ansible 配置和示例 playbook 文件

在此任务中，你将从 GitHub Ansible 配置存储库和示例实验室文件中进行下载。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以确保已安装 **git**：

    ```bash
    sudo apt install git
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以从 GitHub 克隆 ansible 源代码：

    ```bash
    git clone git://github.com/ansible/ansible.git --recursive
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以从 GitHub 克隆 PartsUnlimitedMRP 存储库：

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    >**备注**： 此存储库包含用于创建各种资源的 playbook，我们将在实验室中使用其中一些 playbook。

#### 任务 6：创建和配置 Azure Active Directory 服务主体

在此任务中，你将生成 Azure AD 服务主体，以便于进行访问 Azure 资源所需的 Ansible 的非交互式身份验证。你还需要在上一个任务中创建的资源组上为服务主体分配参与者角色。

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以登录与你的 Azure 订阅关联的 Azure AD 租户：

    ```bash
    az login
    ```

1.  记下显示为上述命令的输出的代码，切换到实验室计算机，从实验室计算机中，在显示 Azure 门户的浏览器窗口中打开另一个选项卡，导航到 [Microsoft 设备登录页](https://microsoft.com/devicelogin)，然后在出现提示时输入该代码并选择 **“下一步”**。

1.  在出现提示时，使用在本实验室中所用的凭据登录，然后关闭浏览器标签页。

1.  切换回 Cloud Shell 窗格的 Bash 会话，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以生成服务主体：

    ```bash
    ANSIBLESP=$(az ad sp create-for-rbac --name az400m1403Ansible)
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以显示新生成的服务主体的属性：

    ```bash
    echo $ANSIBLESP | jq
    ```

1.  查看输出并记录 **appId**、**password** 和 **tenant** 属性的值。输出应采用如下格式：

    ```bash
    {
      "appId": "0df61458-1a6e-4b4c-b02a-d4ae43cd981e",
      "displayName": "az400m1403Ansible",
      "name": "http://az400m1403Ansible",
      "password": "JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U",
      "tenant": "44bb87a9-c9c8-4c0f-b176-88d7814533ba"
    }
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以标识订阅的值：

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    echo $SUBSCRIPTIONID
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，检索基于 Azure 角色的内置访问控制参与者角色的 **ID** 属性值： 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    echo $CONTRIBUTORID
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，在本实验室前面部分创建的资源组上分配参与者角色： 

    ```bash
    APPID=$(echo $ANSIBLESP | jq '.appId' -r)

    RG2NAME=az400m14l03brg
    az role assignment create --assignee "$APPID" \
    --role "$CONTRIBUTORID" \
    --resource-group "$RG2NAME"
    ```

#### 任务 7：配置 Azure AD 凭据和 SSH 以用于 Ansible

在此任务中，你将为 Ansible 配置 Azure 访问权限和用于 Ansible 的 SSH，前者利用在上一个任务中创建的 Azure AD 服务主体。

>**备注**： 若要允许从指定的 Ansible 控制系统进行远程管理，我们可以创建一个凭据文件，也可以将服务主体详细信息导出为 Ansible 环境变量。我们将选择第一个选项。凭据将存储在 **~/.azure/credentials 文件中**。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以创建凭据文件： 

    ```bash
    echo "[default]" > ~/.azure/credentials
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，将包含在上一个任务中标识的 Azure 订阅 ID 的新行追加到 Ansible 凭据文件： 

    ```bash
    echo subscription_id=$SUBSCRIPTIONID >> ~/.azure/credentials
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，将包含在上一个任务中标识的服务主体 ID 的新行追加到 Ansible 凭据文件： 

    ```bash
    echo client_id=$APPID >> ~/.azure/credentials
    ```
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，将包含在上一个任务中标识的服务主体 ID 机密的新行追加到 Ansible 凭据文件： 

    ```bash
    SECRET=$(echo $ANSIBLESP | jq '.password' -r)
    echo secret=$SECRET >> ~/.azure/credentials
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，将包含在上一个任务中标识的 Azure AD 租户 ID 的新行追加到 Ansible 凭据文件： 

    ```bash
    TENANT=$(echo $ANSIBLESP | jq '.tenant' -r)
    echo tenant=$TENANT >> ~/.azure/credentials
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以验证 Ansible 凭据文件的内容： 

    ```bash
    cat  ~/.azure/credentials
    ```

    >**备注**： 输出应如下所示：

    ```bash
    [default]
    subscription_id=1cfb08bc-a39e-46cc-9b1e-38a182b43d60
    client_id=0df61458-1a6e-4b4c-b02a-d4ae43cd981e
    secret=JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U
    tenant=44bb87a9-c9c8-4c0f-b176-88d7814533ba
    ```

    >**备注**： 现在需要创建用于远程 SSH 连接的公钥/私钥对，并测试其操作。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以生成密钥对。在出现提示时，按 **Enter** 键接受文件的默认位置值，无需设置密码：

    ```bash
    ssh-keygen -t rsa
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，授予对托管私钥的 **.ssh** 文件夹的读取、写入和执行权限：

    ```bash
    chmod 755 ~/.ssh
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以创建并设置对 **authorized_keys** 文件的读取和写入权限。

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    >**备注**： 通过提供此文件中包含的密钥，你无需提供密码即可访问此文件。

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以将密码添加到 **authorized_keys** 文件：

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. 在出现提示时，键入 **“是”**，然后为 **azureuser** 用户帐户（之前在本实验室中部署第三个Azure VM 时指定的帐户）输入密码 **Pa55w.rd1234**。 
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以验证系统是否未提示输入密码：

    ```bash
    ssh 127.0.0.1
    ```
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，键入 **“退出”** 并按 **Enter** 键，以终止刚刚建立的环回连接。 

>**备注**： 建立无密码 SSH 身份验证是设置 Ansible 环境的一个关键步骤。 

#### 任务 8：使用 Ansible playbook 创建 Web 服务器 Azure VM

在此任务中，你将使用 Ansible playbook 创建托管 Web 服务器的 Azure VM。 

>**备注**： 在控制 Azure VM 中启动并运行 Ansible 后，我们现在可以部署第一个 playbook，以便创建和配置托管 Azure VM。Playbook 将使用 localhost 文件，而不是动态清单。对于部署，我们将使用示例 playbook **“~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml”**，我们之前在本实验室中从 Github 存储库克隆了该文件。在部署示例 playbook 之前，你需要将包含在其内容中的公共 SSH 密钥替换为在上一个任务中生成的密钥。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，以标识在上一个任务中生成的本地存储的公钥：

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  记录输出，包括输出字符串结尾的用户名。 
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以在 Nano 文本编辑器中打开 **new_vm_web.yml** 文件：

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  在 Nano 编辑器界面中，找到文件末尾的 key_data 值中的 SSH 字符串，然后删除文件中的密钥。
1.  在 Nano 编辑器中找到文件末尾的 SSH 字符串，在 `key_data` 条目中，删除现有的密钥值并将其替换为之前在此任务中记录的密钥值。 
1.  确保文件中包含的 `admin_username` 条目的值与用于登录托管 Ansible 控制系统的 Azure VM 的用户名 (**azureuser**)一致。`Ssh_public_keys` 部分的 `path` 条目必须使用相同的用户名。
1.  将 `vm_size` 条目的值从 `Standard_A0` 更改为 `Standard_D2s_v3`。
1.  如有需要，将 `dnsname: ‘{{ vmname }}.westeurope.cloudapp.azure.com’` 条目中的区域名称更改为要在其中进行部署的 Azure 区域的名称。
1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。

    >**备注**： 我们需要将 Azure VM 部署到实验室开始时创建的资源组中。使用以下值进行部署： 

    | 设置 | 值 |
    | --- | --- |
    | 资源组 | **az400m14l03arg** |
    | 虚拟网络 | **az400m1403vm1VNET** |
    | 子网 | **az400m1403vm1Subnet** |

    >**备注**： 备注：可在 playbook 中定义这些变量，也可在通过包含 `--extra-vars` 选项来调用 `ansible-playbook` 命令时，在运行时输入这些变量。对于 VM 名称，仅使用最多 15 个小写字母和数字（不使用连字符、下划线或大写字母），并确保它是全局唯一的，因为相同名称会用于生成与相应 Azure VM 关联的公共 IP 地址的 DNS 名称。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以部署预配 Azure VM 的示例 ansible playbook（其中的 `<VM_name>` 占位符表示所选的唯一 VM 名称）：

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403vm1VNET subnet=az400m1403vm1Subnet"
    ```

    >**备注**： 如果输入的 VM 名称无效，你可能会收到以下错误消息：

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "The storage account named storageaccountname is already taken. - Reason.already_exists"}`. 要解决此问题，请为 Azure VM 使用其他名称，因为你使用的名称不是全局唯一的。
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "Error creating or updating your-vm-name - Azure Error: InvalidDomainNameLabel\nMessage: The domain name label for your VM is invalid. It must conform to the following regular expression: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$.”}`。要解决此问题，请按照要求的命名约定，为 Azure VM 使用其他名称。 

    >**备注**： 等待部署完成。该操作需要约 3 分钟。 

    >**备注**： 部署完成后，即可运行动态清单，以验证 Ansible 现在可检测到新的 VM。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以生成动态清单：

    ```bash
    python /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py --list | jq
    ```
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以执行 ping 测试，从而验证动态清单文件是否包含新部署的 Azure VM。和之前一样，第一次运行此测试时，你需要验证 SSH 主机密钥，但后续运行不需要任何交互：

    ```bash
    sudo chmod +x /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py
    ansible -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py all -m ping
    ```

    >**备注**： 虽然之前通过 Azure Cloud Shell 部署的两个 Azure VM 都会显示在清单中，但它们都不可访问，因为它们不包含在 Ansible 控制计算机上生成的公钥。收到来自新部署的 Azure VM 的成功响应之后，请继续执行下一个任务。 

1.  如果动态清单未成功完成，请在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，以使用 SSH 连接到新部署的 Azure VM（其中 `<VM_name>` 占位符表示分配给新预配的 Azure VM 的名称）：

    ```bash
    $RG2NAME='az400m14l03arg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  出现确认继续操作的提示时，键入 **“是”** 并按 **Enter** 键，然后在建立连接后，键入 **“退出”** 并再次按 **Enter** 键以终止它。

#### 任务 9：使用 Ansible playbook 配置新部署的 Azure VM

在此任务中，你将运行另一个 Ansible playbook，以配置新创建的计算机。你将使用一个 playbook 来安装软件包 httpd 和从 Github 存储库下载 HTML 页面。完成此操作后，你将有一个功能完善的 Web 服务器。

>**备注**： 我们将使用示例 playbook **“~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml”**。我们将利用变量 **vmname** 来修改 playbook 的主机参数，该参数定义 playbook 的目标主机（动态清单脚本返回的主机除外）。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，以验证新部署的 Azure VM 当前是否未运行任何 Web 服务（其中的 `<IP_address>` 占位符表示分配给新预配的 Azure VM 的网络适配器的公共 IP 地址）：

    ```bash
    curl http://<IP_address>
    ```

    >**备注**： 可通过运行以下脚本来标识此公共 IP 地址（其中的 `<VM_name>` 占位符表示分配给新预配的 Azure VM 的名称）：

    ```bash
    $RG2NAME='az400m14l03brg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    echo $PIP
    ```

1.  验证响应的格式是否为 `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused`。
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，以使用 Ansible playbook 安装 HTTP 服务（其中的 `<VM_name>` 占位符表示分配给新预配的 Azure VM 的名称）： 

    ```bash
    ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"
    ```

    >**备注**： 请等待安装完成。所需时间应该不超过一分钟。 

1.  安装完成后，在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，以验证新部署的 Azure VM 当前是否正在运行 Web 服务（其中的 `<IP_address>` 占位符表示分配给新预配的 Azure VM 的网络适配器的公共 IP 地址）：

    ```bash
    curl http://<IP_address>
    ```

    >**备注**： 输出应具有以下内容： 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

#### 任务 10：使用 Ansible playbook 在 Azure 中实现配置管理和预期状态

在此任务中，你将使用 Ansible playbook 在 Azure 中实现配置管理和预期状态。

>**备注**： 我们可以定期运行 `ansible-playbook` 命令，以确保配置与相应的 playbook 的内容保持一致。为了以自动化方式完成此操作，我们将利用 Linux cron 功能。在此任务中，我们将每隔一分钟运行一次命令，但是在生产环境中，你可能会选择更低的频率。

>**备注**： 我们将使用 Ansible playbook 来设置 cron 作业。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以在 Nano 文本编辑器中打开 Ansible 配置文件：

    ```bash
    sudo nano /etc/ansible/ansible.cfg
    ```

1.  在 Nano 编辑器界面的 `[Default]` 部分，删除 `#host_key_checking = False` 行中的前导哈希字符 `#`
1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以在 Nano 文本编辑器中打开配置 cron 作业的 playbook：

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  在 Nano 编辑器界面的 `job` 条目中，将 `< your-vm-name >` 占位符替换为在上一个任务中配置的 Azure VM 的名称。 
1.  在 Nano 编辑器界面中，将 `job` 条目设置为 `ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  /home/azureuser/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars “vmname=<VM_name>”`，其中的 `<VM_name>` 占位符表示分配给托管的 Azure VM 的名称。
1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以创建 cron 文件：

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以验证 cron 文件是否正在运行：

    ```bash
    sudo tail /var/log/syslog
    ```

    >**备注**： 你需要根目录特权，因为所有用户的所有 cron 作业均记录在同一文件中。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令以从 cron 编辑器界面识别 cron 文件：

    ```bash
    crontab -e
    ```

1.  在出现提示时，选择 **1** 表示 Nano，然后按 **Enter** 键。
1.  在 **crontab** 编辑器中，验证是否存在 **“usr/bin/ansible-playbook”** 条目，然后按 **ctrl + x**组合键以关闭编辑器。

    >**备注**： 5 个星号表示作业每分钟运行一次。

    >**备注**： 现在验证设置是否有效。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，运行以下命令，以通过 SSH 连接到托管的 Azure VM（其中的 `<IP_address>` 占位符表示分配给该 Azure VM 的网络适配器的公共 IP 地址）：

    ```bash
    ssh azureuser@<IP_address>
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以验证网站是否仍在运行：

    ```bash
    curl http://127.0.0.1
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以删除网站的主页：

    ```bash
    rm /var/www/html/index.html
    ```

1.  等待一分钟，然后在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以验证网站的主页是否已还原：

    ```bash
    curl http://127.0.0.1
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，键入 **“退出”** 并按 **Enter** 键以返回到 Ansible 控制计算机的 SSH 会话。

    >**备注**： 通过定期运行 ansible playbook，可纠正改变 Azure VM 预期配置的状态的更改（如 playbook 中的定义所示）。使用此方法，我们可以防止配置偏移并保留预期状态。我们还可以采用相同的方法并定期运行面向 Azure 资源的 Ansible playbook。这将帮助你确保按预期部署和配置基础结构。 

#### 任务 11：结合使用 Ansible 和 Azure 资源管理器模板，促进 Azure 资源的配置和预期状态管理

在此任务中，你将结合使用 Ansible 和 Azure 资源管理器模板，促进 Azure 资源的配置和预期状态管理。

>**备注**： 如上一个任务中所述，Ansible 可用于修复相应模块支持的现有资源的配置偏差。但是，你也可以使用 Ansible 来部署将引用 Azure 资源管理器模板的 playbook，它提供直接访问 Azure 资源管理器提供的功能和资源的权限。 

>**备注**： 我们将部署另一个 Azure VM，但是目前我们将使用引用 Azure 资源管理器模板的 ansible playbook。为了简单起见，我们将使用[一种非常简洁的 Azure 快速启动模板来预配单个存储帐户](https://github.com/Azure/azure-quickstart-templates/tree/master/101-storage-account-create)。可通过查看 **“PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml”** playbook 来标识相关的 playbook 语法。

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令，在 Nano 文本编辑器中打开调用 Azure 资源管理器模板的 playbook：

    ```bash
    nano  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml
    ```

1.  在 Nano 编辑器界面的 `templateLink:` 条目中，将当前 URL 替换为 `https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json`。
1.  在 Nano 编辑器界面中，按 **ctrl + o** 组合键，再按 **Enter** 键，然后按 **ctrl + x** 组合键保存更改并关闭文件。
1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以执行 playbook（将 `<Azure_region>` 占位符替换为本实验室中在其中部署所有资源的 Azure 区域的名称）：

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    >**备注**：ARM 模板部署是幂等的，这意味着部署一次 ARM 模板与多次部署该模板的效果相同，了解这一点非常重要。换言之，你可以安全地将 ARM 模板重新部署到同一资源组，而无需担心会创建重复的资源。实际上，你可以计划定期重新部署 ARM 模板，例如借助 Linux cron 实用工具，如上一个任务中所述。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以执行相同的 playbook（将 `<Azure_region>` 占位符替换为本实验室中在其中部署所有资源的 Azure 区域的名称）。

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    >**备注**： 现在我们需要修改通过模板部署的存储帐户。此类更改可能难以检测，但可能会对环境造成不利影响。因此，拥有一个可自动将这些更改还原为其预期状态的机制非常有用。在本例中，我们将更改存储帐户的复制设置。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以标识之前在本实验室中部署的存储帐户的当前设置：

    ```bash
    RGNAME=az400m14l03brg
    STORAGEACCOUNTNAME=$(az storage account list --resource-group $RGNAME --query "[].name" --output tsv)
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令，以将 SKU 从 `Standard_LRS` 更改为 `Standard_GRS`并验证是否已进行更改： 

    ```bash
    az storage account update \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --sku 'Standard_GRS'

    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

    >**备注**：现在重新运行部署相同模板的 playbook。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以重新运行相同的 playbook（将 `<Azure_region>` 占位符替换为本实验室中在其中部署所有资源的 Azure 区域的名称）：

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，在配置为 Ansible 控制计算机的 Azure VM 的 SSH 会话中，在配置为 Web 服务器的 Azure VM 的 SSH 会话中，运行以下命令以验证是否已还原更改： 

    ```bash
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

### 练习 3：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**： 请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**： 该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 回顾

在本实验室中，你已了解如何使用 Ansible 部署、配置和管理 Azure 资源。 

