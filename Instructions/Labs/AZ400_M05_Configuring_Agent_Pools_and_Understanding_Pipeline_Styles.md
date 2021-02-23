---
lab:
    title: '实验室：配置代理池并了解管道样式'
    az400Module: '模块 5：配置 Azure Pipelines'
---

# 实验室：配置代理池并了解管道样式
# 学生实验室手册

## 实验室概述

通过基于 YAML 的管道，你能够以代码形式完全实现 CI/CD，其中管道定义与 Azure DevOps 项目的代码驻留在同一存储库中。基于 YAML 的管道支持经典管道的多种功能，例如拉取请求、代码评审、历史记录、分支和模板。 

无论选择哪种管道样式，都需要代理才能使用 Azure Pipelines 生成代码或部署解决方案。代理托管一次运行一项作业的计算资源。作业可直接在代理的主机上运行，也可在容器中运行。你可以选择使用 Microsoft 为你托管的托管代理来运行作业，也可以实现你自己设置和管理的自托管代理。 

在本实验室中，你将逐步完成以下过程：将经典管道转换为基于 YAML 的管道，并先使用 Microsoft 托管代理运行该管道，然后使用自托管代理再次运行该管道。

## 目标

完成本实验室后，你将能够：

- 实现基于 YAML 的管道
- 实现自托管代理

## 实验室持续时间

-   预计用时：**90 分钟**

## 说明

### 准备工作

#### 登录实验室虚拟机

确保已使用以下凭据登录到 Windows 10 计算机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看已安装的应用程序

在你的 Windows 10 桌面上找到任务栏。任务栏里有本实验室中你将使用的应用程序的图标：
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/)。此应用程序将作为本实验室先决条件安装。

#### 设置 Azure DevOps 组织

如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/zh-cn/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

#### 设置 GitHub 帐户

按照[注册一个新的 GitHub 帐户](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account)中的说明进行操作。

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板的预配置的 Parts Unlimited 团队项目。

#### 任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 **PartsUnlimited** 模板生成一个新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **备注**：有关该站点的详细信息，请参阅 https://docs.microsoft.com/zh-cn/azure/devops/demo-gen。

1.  单击 **“登录”**，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录*。
1.  如果需要，在 **“Azure DevOps 演示生成器”** 页面上，单击 **“接受”** 以接受访问 Azure DevOps 订阅的权限请求。
1.  在 **“新建项目”** 页面上的“新建项目名称”文本框中，键入 **“配置代理池并了解管道样式”**，在 **“选择组织”** 下拉列表中选择你的 Azure DevOps 组织，然后单击 **“选择模板”**。
1.  在 **“选择模板”** 页面上，单击 **“PartsUnlimited”** 模板，然后单击 **“选择模板”**。
1.  返回 **“新建项目”** 页面，选中 **“ARM 输出”** 标签下方的复选框，然后单击 **“创建项目”**

    > **备注**：等待此过程完成。该过程需要约 2 分钟。如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在 **“新建项目”** 页面上，单击 **“导航到项目”**。

#### 任务 2：安装 Visual Studio Code

在此任务中，你将安装 Visual Studio Code。如果你已实现这些先决条件，则可直接继续执行下一个任务。

1.  如果尚未安装 Visual Studio Code，请从 Web 浏览器窗口导航到 [Visual Studio Code 下载页面](https://code.visualstudio.com/)以进行下载和安装。 

#### 任务 3：准备用于映像部署的实验室环境

在此任务中，你将准备用于部署映像的实验室环境，该映像将用于预配自托管代理。

> **备注**：自托管代理及其配置的位置很大程度上取决于你的需求。在本练习中，你将为此使用 Azure VM，该 VM 基于用于 Microsoft 托管代理的同一映像。可从[虚拟环境 GitHub 存储库](https://github.com/actions/virtual-environments)中获取映像。还可在此处找到 GitHub Actions 托管的运行器的映像。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [GitHub](https://github.com) 并登录你的 GitHub 帐户。
1.  在显示你的 GitHub 帐户的 Web 浏览器中，单击右上角表示配置文件的圆形图标，然后在下拉菜单中单击 **“设置”**。
1.  在 **“公用配置文件”** 页面上，在垂直菜单左侧，单击 **“开发人员设置”**。
1.  在 **“GitHub 应用”** 页面上，在垂直菜单左侧，单击 **“个人访问令牌”**。
1.  在 **“个人访问令牌”** 页面上，单击右上角的 **“生成新令牌”**。
1.  在 **“新的个人访问令牌”** 页面上，在“备注”文本框中键入 **“az400m05l05a 实验室”**，在 **“选择范围”** 部分选中 **“read:packages”** 复选框，然后单击 **“生成令牌”**。
1.  在 **“个人访问令牌”** 页面上，标识新生成的令牌并记录其值。你将在此任务的稍后部分使用它。

    > **备注**：此时请务必记录个人访问令牌。导航离开当前页面之后，你将不能再检索其值。 

1.  在实验室计算机上，启动 Web 浏览器，导航到 [Packer 下载页面](https://www.packer.io/downloads)，下载当前版本的 Windows 64 位版本的 Packer，打开包含 **packer.exe** 二进制文件的存储，然后将其提取到 **C:\Windows** 目录。 
1.  在实验室计算机上，以管理员身份启动 Windows PowerShell ISE，然后从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令安装 Chocolatey：

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```

1.  从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令安装 git：

    ```powershell
    choco install git -params '"/GitOnlyOnPath"' -y
    ```

1.  从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令安装 packer：

    ```powershell
    choco install packer -y
    ```

1.  从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令安装 Azure CLI：

    ```powershell
    Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
    ```

1.  从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令安装 PowerShell（出现确认继续操作的提示时，单击 **“全部确定”**）：

    ```powershell
    Install-Module -Name Az
    ```

1.  从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令登录 Azure 订阅（系统提示提供凭据时，使用 Microsoft 帐户或 Azure AD 帐户进行身份验证，后者具有 Azure 订阅中“所有者”角色和与该 Azure 订阅相关联的 Azure AD 租户中“全局管理员”角色）：

    ```powershell
    Add-AzAccount
    ```

1.  从 **“管理员: 窗口的控制台”** 窗格中，运行以下命令安装 git：

    ```powershell
    New-Item -Type Directory -Path 'C:\Labfiles' -Force
    Set-Location c:\Labfiles
    git clone https://github.com/actions/virtual-environments.git virtual-environments -q
    ```

1.  在实验室计算机上，启动文件资源管理器并导航到 **C:\\Labfiles\\virtual-environments\\images\\win** 目录，在记事本中打开 **“windows2019.json”** 文件，并查看其内容。

    > **备注**：此文件定义映像的内容。可以自定义其配置，并通过修改 **“配置程序”** 部分有效地影响映像预配时间。

1.  在实验室计算机上，在文件资源管理器中，导航到 **C:\\Labfiles\\virtual-environments\\helpers** 目录，并在记事本中打开 **“GenerateResourcesAndImage.ps1”** 文件。 
1.  在显示 **“GenerateResourcesAndImage.ps1”** 文件内容的记事本中，在包含 `New-AzStorageAccount -ResourceGroupName $ResourceGroupName -AccountName $storageAccountName -Location $AzureLocation -SkuName "Standard_LRS"``, replace `"Standard_LRS"` 和 `"Premium_LRS"` 命令的行中，保存更改并关闭文件。

    > **备注**：此更改旨在加快映像预配速度。

1.  在实验室计算机上，在文件资源管理器中，导航到 **C:\\Labfiles\\virtual-environments\\images\\win** 目录，在记事本中打开 **“windows2016.json”** 文件，粘贴以下内容，替换现有内容，保存更改并关闭文件。

    > **备注**：做出这些更改是为了在此特定实验室方案中严格加快映像预配的速度。一般说来，应调整映像以满足需求。 

    ```json
    {
        "variables": {
            "client_id": "{{env `ARM_CLIENT_ID`}}",
            "client_secret": "{{env `ARM_CLIENT_SECRET`}}",
            "subscription_id": "{{env `ARM_SUBSCRIPTION_ID`}}",
            "tenant_id": "{{env `ARM_TENANT_ID`}}",
            "object_id": "{{env `ARM_OBJECT_ID`}}",
            "resource_group": "{{env `ARM_RESOURCE_GROUP`}}",
            "storage_account": "{{env `ARM_STORAGE_ACCOUNT`}}",
            "temp_resource_group_name": "{{env `TEMP_RESOURCE_GROUP_NAME`}}",
            "location": "{{env `ARM_RESOURCE_LOCATION`}}",
            "virtual_network_name": "{{env `VNET_NAME`}}",
            "virtual_network_resource_group_name": "{{env `VNET_RESOURCE_GROUP`}}",
            "virtual_network_subnet_name": "{{env `VNET_SUBNET`}}",
            "private_virtual_network_with_public_ip": "{{env `PRIVATE_VIRTUAL_NETWORK_WITH_PUBLIC_IP`}}",
            "vm_size": "Standard_D8s_v3",
            "run_scan_antivirus": "false",
            "root_folder": "C:",
            "toolset_json_path": "{{env `TEMP`}}\\toolset.json",
            "image_folder": "C:\\image",
            "imagedata_file": "C:\\imagedata.json",
            "helper_script_folder": "C:\\Program Files\\WindowsPowerShell\\Modules\\",
            "psmodules_root_folder": "C:\\Modules",
            "install_user": "installer",
            "install_password": null,
            "capture_name_prefix": "packer",
            "image_version": "dev",
            "image_os": "win16"
        },
        "sensitive-variables": [
            "install_password",
            "client_secret"
        ],
        "builders": [
            {
                "name": "vhd",
                "type": "azure-arm",
                "client_id": "{{user `client_id`}}",
                "client_secret": "{{user `client_secret`}}",
                "subscription_id": "{{user `subscription_id`}}",
                "object_id": "{{user `object_id`}}",
                "tenant_id": "{{user `tenant_id`}}",
                "os_disk_size_gb": "256",
                "location": "{{user `location`}}",
                "vm_size": "{{user `vm_size`}}",
                "resource_group_name": "{{user `resource_group`}}",
                "storage_account": "{{user `storage_account`}}",
                "temp_resource_group_name": "{{user `temp_resource_group_name`}}",
                "capture_container_name": "images",
                "capture_name_prefix": "{{user `capture_name_prefix`}}",
                "virtual_network_name": "{{user `virtual_network_name`}}",
                "virtual_network_resource_group_name": "{{user `virtual_network_resource_group_name`}}",
                "virtual_network_subnet_name": "{{user `virtual_network_subnet_name`}}",
                "private_virtual_network_with_public_ip": "{{user `private_virtual_network_with_public_ip`}}",
                "os_type": "Windows",
                "image_publisher": "MicrosoftWindowsServer",
                "image_offer": "WindowsServer",
                "image_sku": "2016-Datacenter",
                "communicator": "winrm",
                "winrm_use_ssl": "true",
                "winrm_insecure": "true",
                "winrm_username": "packer"
            }
        ],
        "provisioners": [
            {
                "type": "powershell",
                "inline": [
                    "New-Item -Path {{user `image_folder`}} -ItemType Directory -Force"
                ]
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/ImageHelpers",
                "destination": "{{user `helper_script_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/SoftwareReport",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/post-generation",
                "destination": "C:/"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/Tests",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{template_dir}}/toolsets/toolset-2016.json",
                "destination": "{{user `toolset_json_path`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "net user {{user `install_user`}} {{user `install_password`}} /add /passwordchg:no /passwordreq:yes /active:yes /Y",
                    "net localgroup Administrators {{user `install_user`}} /add",
                    "winrm set winrm/config/service/auth @{Basic=\"true\"}",
                    "winrm get winrm/config/service/auth"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if (-not ((net localgroup Administrators) -contains '{{user `install_user`}}')) { exit 1 }"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "ImageVersion={{user `image_version`}}",
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}",
                    "PSMODULES_ROOT_FOLDER={{user `psmodules_root_folder`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-PowerShellModules.ps1",
                    "{{ template_dir }}/scripts/Installers/Initialize-VM.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-WebPlatformInstaller.ps1"
                ],
                "execution_policy": "unrestricted"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-DotnetTLS.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "inline": [
                    "setx ImageVersion {{user `image_version` }} /m",
                    "setx ImageOS {{user `image_os` }} /m"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "IMAGE_VERSION={{user `image_version`}}",
                    "IMAGEDATA_FILE={{user `imagedata_file`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-ImageData.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-PowershellCore.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "valid_exit_codes": [
                    0,
                    3010
                ],
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-VS.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-NET48.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
                {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Nuget.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Vsix.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-NodeLts.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-7zip.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Packer.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-OpenSSL.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Git.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-GitHub-CLI.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Enable-DeveloperMode.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "wmic product where \"name like '%%microsoft azure powershell%%'\" call uninstall /nointeractive"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Cmake.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-GitVersion.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-DynamicPort.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Finalize-VM.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-Antivirus.ps1",
                    "{{ template_dir }}/scripts/Installers/Disable-JITDebugger.ps1"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if( Test-Path $Env:SystemRoot\\System32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\System32\\Sysprep\\unattend.xml -Force}",
                    "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
                    "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
                ]
            }
        ]
    }
    ```

1.  在实验室计算机上，在文件资源管理器中，导航到 **C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Installers** 目录，在记事本中打开 **“VisualStudio.Tests.ps1”** 文件，找到以下部分：

    ```powershell
    $workLoads = @(
	"--allWorkloads --includeRecommended"
	$requiredComponents
	"--remove Component.CPython3.x64"
    )
    ```

1.  在显示 **“VisualStudio.Tests.ps1”** 文件内容的记事本窗口中，将上一步中定义的部分替换为以下内容：

    ```powershell
    $workLoads = @(
	"--add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio --includeRecommended"
	"--remove Component.CPython3.x64"
    )
    ```

    > **备注**：进行此更改是为了在此特定实验室方案中严格加快映像预配的速度。一般说来，应调整要安装的 Visual Studio 组件以满足需求。 

1.  在实验室计算机上，在文件资源管理器中，导航到  **C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Tests** 目录，在记事本中打开 **“VisualStudio.Tests.ps1”** 文件，将行 `$expectedComponents = Get-ToolsetContent | Select-Object -ExpandProperty visualStudio | Select-Object -ExpandProperty workloads` 替换为 `$expectedComponents = "Microsoft.VisualStudio.Workload.NetWeb"`，保存更改并关闭文件。

    > **备注**：进行此更改是为了在此特定实验室方案中严格加快映像预配的速度。一般说来，应调整测试以满足需求。 

1.  切换回 **“管理员:窗口，从其控制台** 窗格运行以下命令，以导入 Packer PowerShell 模块，你将使用该模块生成操作系统映像，该映像将用于预配自托管代理：

    ```powershell
    Set-Location c:\Labfiles\virtual-environments
    Import-Module .\helpers\GenerateResourcesAndImage.ps1
    ```

1.  从 **“管理员: 窗口的脚本窗格中**，运行以下命令以使用 Packer PowerShell 模块生成操作系统映像，该映像将用于预配预配自托管代理（将 `<Azure_region>` 占位符替换为要在其中部署 Azure VM 的 Azure 区域的名称，将 `<GitHub_PAT>` 占位符替换为之前在此任务中生成的 GitHub 个人访问令牌的值）：

    ```powershell
    $location = '<Azure_region>'
    $githubPAT = '<GitHub_PAT>'
    $random = (-join (((48..57)+(65..90)+(97..122)) * 80 | Get-Random -Count 6 |%{[char]$_})).ToLower()
    $rgName = "az400m05$random-RG"
    $subscriptionId = (Get-AzSubscription).SubscriptionId  
    GenerateResourcesAndImage -SubscriptionId $subscriptionId -ResourceGroupName $rgName -ImageGenerationRepositoryRoot "$pwd" -ImageType 'Windows2016' -AzureLocation $location -GitHubFeedToken $githubPAT
    ```

1.  出现提示时，使用你之前使用的同一用户帐户进行登录，以对与 Azure 订阅相关联的 Azure AD 租户进行身份验。

    > **备注**：请勿等待脚本完成，而是继续进行下一个练习。预配映像可能需要 60 分钟左右。

### 练习 1：创作基于 YAML 的 Azure DevOps 管道

在本练习中，你需要将经典 Azure DevOps 管道转换为基于 YAML 的管道。 

#### 任务 1：创建 Azure DevOps YAML 管道

在此任务中，你将创建基于模板的 Azure DevOps YAML 管道。

1.  在 **“配置代理池并了解管道样式”** 项目处于打开状态时，从显示 Azure DevOps 门户的 Web 浏览器中，在垂直导航窗格的左侧，单击 **“管道”**。 
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，单击 **“新建管道”**。
1.  在 **“你的代码在哪里?”** 窗格上，单击 **“Azure Repos Git”**。 
1.  在 **“选择存储库”** 窗格上，单击 **“PartsUnlimited”**。
1.  在 **“配置你的管道”** 窗格上，单击 **“初学者管道”**。
1.  在 **“查看你的管道 YAML”** 窗格上，查看示例管道，单击 **“保存并运行”** 按钮旁边方向朝下的插入符号，单击 **“保存”**，然后在 **“保存”** 窗格上，单击 **“保存”**。

    > **备注**：这会在托管项目代码的存储库的根目录中创建 **“azure-pipelines.yml”** 文件。

    > **备注**：需要将示例 YAML 管道的内容替换为经典编辑器生成的管道代码，并对其进行修改，以说明经典管道和 YAML 管道之间的差异。

#### 任务 2：将经典管道转换为 YAML 管道

在此任务中，你需要将经典管道转换为 YAML 管道

1.  在 **“配置代理池并了解管道样式”** 项目处于打开状态时，从显示 Azure DevOps 门户的 Web 浏览器中，在垂直导航窗格的左侧，单击 **“管道”**。 
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，将鼠标指针悬停在包含 **“PartsUnlimitedE2E”** 条目的条目最右侧，以显示指示 **“更多”** 菜单的垂直省略号，单击省略号，然后在下拉列表中单击 **“编辑”**。这会显示在实验室开始时生成的项目的生成管道。 
1.  在 **“PartsUnlimitedE2E”** 编辑窗格的 **“任务”** 选项卡上，单击 **“触发器”**，在 **“PartsUnlimited”** 窗格的右侧，取消选中 **“启用持续集成”** 复选框，单击 **“保存并排队”** 按钮旁边朝下的插入符号，在下拉菜单中单击 **“保存”**，然后在 **“保存生成管道”** 中单击 **“保存”**。

    > **备注**：这会防止出现由于在本实验室期间对存储库的更改而意外执行自动生成的情况。

    > **备注**：或者在将现有管道的内容复制到新管道之后直接删除它。

1.  在 Azure DevOps 门户中，在垂直导航窗格左侧，单击 **“管道”** 部分的 **“管道”**。 
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，单击 **“PartsUnlimitedE2E”** 条目。 
1.  在 **“PartsUnlimitedE2E”** 窗格的“运行”选项卡上，单击右上角的垂直省略号，然后在下拉菜单中单击 **“导出为 YAML”**。这会自动将 **“build.yml”** 文件下载到你的本地 **“下载”** 文件夹。

    > **备注**：**“导出为 YAML”** 功能将替换之前的 **“查看 YAML”** 选项，该选项从 Azure DevOps 门户中的管道编辑器窗格提供，并且存在限制，一次只能查看一个作业的 YAML 内容。新功能利用现有的经典管道和 YAML 管道基础结构，包括 YAML 分析库，这有助于在两种管道之间进行更准确的转换。它支持以下管道组件：

    - 单个和多个作业
    - 签出选项
    - 执行计划并行
    - 超时和名称元数据
    - 需求
    - 计划和其他触发器
    - 池选择，包括不同于默认设置的作业
    - 所有任务和所有输入，包括优化默认输入
    - 作业和步骤条件
    - 任务组展开

    > **备注**：新功能未涵盖的组件只有变量和时区转换。采用 YAML 定义的变量会替代在 Azure DevOps 门户的用户界面设置的变量。如果 **“导出为 YAML”** 功能检测到存在经典管道变量，它们将显式包含在新生成的 YAML 管道定义内的注释中。同样，由于采用 YAML 格式的 cron 计划的表示形式为 UTC，因此尽管经典计划依赖于组织的时区，但是它们也包含在注释中。

    > **备注**：有关此功能的详细信息，请参阅[替换“查看 YAML”](https://devblogs.microsoft.com/devops/replacing-view-yaml/)

1.  在实验室计算机上，启动 Visual Studio Code，并使用它打开文件 **“build.yml”**。该文件应具有以下内容：

    ```yaml
    name: $(date:yyyyMMdd)$(rev:.r)
    jobs:
    - job: Phase_1
      displayName: Phase 1
      cancelTimeoutInMinutes: 1
      pool:
        vmImage: vs2017-win2016
      steps:
      - checkout: self
      - task: NuGetInstaller@0
        name: NuGetInstaller_1
        displayName: NuGet restore
        inputs:
          solution: '**\*.sln'
      - task: VSBuild@1
        name: VSBuild_2
        displayName: Build solution
        inputs:
          vsVersion: 15.0
          msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: VSTest@1
        name: VSTest_3
        displayName: Test Assemblies
        inputs:
          testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
          codeCoverageEnabled: true
          vsTestVersion: latest
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: CopyFiles@2
        name: CopyFiles1
        displayName: Copy Files
        inputs:
          SourceFolder: $(build.sourcesdirectory)
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  在 Visual Studio Code 窗口中，单击顶级菜单中的 **“选择”**，然后在下拉菜单中单击 **“全选”**。
1.  在 Visual Studio Code 窗口中，单击顶级菜单中的 **“编辑”**，然后在下拉菜单中单击 **“复制”**。
1.  切换到显示 Azure DevOps 门户的浏览器窗口，在垂直导航窗格左侧，单击 **“管道”** 部分的 **“管道”**。 
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，选择 **“PartsUnlimited”**，然后在 **“PartsUnlimited”** 窗格上，选择 **“编辑”**。
1.  在 **“PartsUnlimited”** 编辑窗格上，选择管道的现有 YAML 内容，将其替换为剪贴板的内容。
1.  在 **“PartsUnlimited”** 编辑窗格上，单击右上角的 **“保存”**，然后在 **“保存”** 窗格上，单击 **“保存”**。这将自动触发基于此管道的生成。 
1.  在 Azure DevOps 门户中，在垂直导航窗格左侧，单击 **“管道”** 部分的 **“管道”**。
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，单击 **“PartsUnlimited”** 条目，在 **“PartsUnlimited”** 窗格的“运行”选项卡上，选择最近的运行，在该运行的 **“摘要”** 窗格上，向下滚动至底部，在 **“作业”** 部分单击 **“阶段 1”**，并监视作业，直到作业成功完成。 

### 练习 2：管理 Azure DevOps 代理池

在本练习中，你将实现自托管的 Azure DevOps 代理。

> **备注**：开始本练习之前，请确保在本实验室开始时启动的映像预配脚本已顺利完成。

#### 任务 1：部署基于 Azure VM 的自托管代理

在此任务中，你将通过使用在本实验室开始时生成的自定义映像，部署将用于预配自托管代理的 Azure VM。

1.  在实验室计算机上，切换到 **“管理员:窗口，查看在本实验**室开始时调用的 **GenerateResourcesAndImage** 脚本的输出，并通过检查 **OSDiskUri** 字符串（标识生成的 vhd 文件的完整路径）的第一部分来标识存储帐户名称的值。

    > **备注**：输出应类似于以下值：`OSDiskUri: https://az400m05l05xrg001.blob.core.windows.net/system/Microsoft.Compute/Images/images/packer-osDisk.40096600-7cac-47f4-8573-ffaa113c78b1.v
hd'，其中 `az400m05l05xrg001` 表示存储帐户名称。

1.  从实验室计算机启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用在上一个任务中用于预配 Azure VM 的用户帐户登录。 
1.  在 Azure 门户中，搜索并选择 **“存储帐户”** 资源类型，然后在 **“存储帐户”** 边栏选项卡上，单击之前在此任务中标识的存储帐户的名称。 
1.  在“存储帐户”边栏选项卡上，记下包含该存储帐户的资源组的名称。你将在此任务的稍后部分使用它。
1.  在“存储帐户”边栏选项卡上，在垂直菜单左侧，单击 **“BLOB 服务”** 部分的 **“容器”**。
1.  在“容器”边栏选项卡上，单击 **“+ 容器”**，在 **“新建容器”** 边栏选项卡上，在 **“名称”** 文本框中键入 **“磁盘”**，确保 **“公共访问级别”** 下拉框包含 **“专用(拒绝匿名访问)”** 条目，然后单击 **“创建”**。
1.  返回到“容器”边栏选项卡，在容器列表中，单击 **“映像”**，然后在 **“映像”** 边栏选项卡上，单击表示新生成的映像的条目。
1.  在显示映像属性的边栏选项卡上，单击 **“URL”** 条目旁边的 **“复制到剪贴板”** 图标，并将复制的值粘贴到记事本。你在下一个步骤中将使用它。 
1.  在实验室计算机上，切换到 **“管理员: 窗口，然后从其脚本** 窗格中，通过使用 Azure 资源管理器模板运行以下命令，以设置部署 Azure VM 所需的变量（将 `<resource_group_name>` 占位符替换为之前在此任务中标识的资源组的名称，将 `<storage_account_name>` 占位符替换为之前在此任务中标识的存储帐户的名称，并将 `<image_url>` 占位符替换为之前在此任务中标识的映像 URL 的名称）：

    ```powershell
    $rgName = `<resource_group_name>'
    $storageAccountName = '<storage_account_name>'
    $imageUrl = '<image_url>'
    ```

1.  在实验室计算机上，切换到文件资源管理器窗口，使用以下内容在 **C:\\Labfiles** 文件夹中创建一个名为 **“az400m05-vm0.deployment.template.json”** 新文件，然后保存该文件并将其关闭。 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "vmSize": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
              "description": "Virtual machine size"
            }
          },
          "adminUsername": {
            "type": "string",
            "metadata": {
              "description": "Admin username"
            }
          },
          "adminPassword": {
            "type": "securestring",
            "metadata": {
              "description": "Admin password"
            }
          },
          "storageAccountName": {
            "type": "string",
            "metadata": {
               "description": "storage account name"
            }
          },
          "imageUrl": {
            "type": "string",
            "metadata": {
               "description": "image Url"
            }
          },
          "diskContainer": {
            "type": "string",
            "defaultValue": "disks",
            "metadata": {
              "description": "disk container"
            }
          }
        },
      "variables": {
        "vmName": "az400m05-vm0",
        "osType": "Windows",
        "osDiskVhdName": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/',parameters('diskContainer'),'/',variables('vmName'),'-osDisk.vhd')]",
        "nicName": "az400m05-vm0-nic0",
        "virtualNetworkName": "az400m05-vnet0",
        "publicIPAddressName": "az400m05-vm0-nic0-pip",
        "nsgName": "az400m05-vm0-nic0-nsg",
        "vnetIpPrefix": "10.0.0.0/22", 
        "subnetIpPrefix": "10.0.0.0/24", 
        "subnetName": "subnet0",
        "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
        "computeApiVersion": "2018-06-01",
        "networkApiVersion": "2018-08-01"
      },
        "resources": [
            {
                "name": "[variables('vmName')]",
                "type": "Microsoft.Compute/virtualMachines",
                "apiVersion": "[variables('computeApiVersion')]",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[variables('nicName')]"
                ],
                "properties": {
                    "osProfile": {
                        "computerName": "[variables('vmName')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]",
                        "windowsConfiguration": {
                            "provisionVmAgent": "true"
                        }
                    },
                    "hardwareProfile": {
                        "vmSize": "[parameters('vmSize')]"
                    },
                    "storageProfile": {
                        "osDisk": {
                            "name": "[concat(variables('vmName'),'-osDisk')]",
                            "osType": "[variables('osType')]",
                            "caching": "ReadWrite",
                            "image": {
                                "uri": "[parameters('imageUrl')]"
                            },
                            "vhd": {
                                "uri": "[variables('osDiskVhdName')]"
                            },
                            "createOption": "fromImage"
                        },
                        "dataDisks": []
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "properties": {
                                    "primary": true
                                },
                                "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                            }
                        ]
                    }
                }
            },
            {
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[variables('virtualNetworkName')]",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Virtual Network",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "[variables('vnetIpPrefix')]"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnetName')]",
                            "properties": {
                                "addressPrefix": "[variables('subnetIpPrefix')]"
                            }
                        }
                    ]
                }
            },
            {
                "name": "[variables('nicName')]",
                "type": "Microsoft.Network/networkInterfaces",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Primary NIC",
                "dependsOn": [
                    "[variables('publicIpAddressName')]",
                    "[variables('nsgName')]",
                    "[variables('virtualNetworkName')]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "subnet": {
                                    "id": "[variables('subnetRef')]"
                                },
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIpAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIpAddresses', variables('publicIpAddressName'))]"
                                }
                            }
                        }
                    ],
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
                    }
                }
            },
            {
                "name": "[variables('publicIpAddressName')]",
                "type": "Microsoft.Network/publicIpAddresses",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Public IP for Primary NIC",
                "properties": {
                    "publicIpAllocationMethod": "Dynamic"
                }
            },
            {
                "name": "[variables('nsgName')]",
                "type": "Microsoft.Network/networkSecurityGroups",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Network Security Group (NSG) for Primary NIC",
                "properties": {
                    "securityRules": [
                        {
                            "name": "default-allow-rdp",
                            "properties": {
                                "priority": 1000,
                                "sourceAddressPrefix": "*",
                                "protocol": "Tcp",
                                "destinationPortRange": "3389",
                                "access": "Allow",
                                "direction": "Inbound",
                                "sourcePortRange": "*",
                                "destinationAddressPrefix": "*"
                            }
                        }
                    ]
                }
            }
        ],
        "outputs": {}
    }
    ```

1.  在实验室计算机上，在文件资源管理器窗口，使用以下内容在 **C:\\Labfiles** 文件夹中创建一个名为 **“az400m05-vm0.deployment.template.parameters.json”** 新文件，然后保存该文件并将其关闭。 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vmSize": {
                "value": "Standard_D2s_v3"
            },
            "adminUsername": {
                "value": "Student"
            },
            "adminPassword": {
                "value": "Pa55w.rd1234"
            }
        }
    }
    ```    

1.  从 **“管理员: 窗口，通过使用在本**实验室开始时生成的自定义映像，并运行以下命令部署将用于预配自托管代理的 Azure VM：

    ```powershell
    Set-Location -Path 'C:\Labfiles'
    New-AzResourceGroupDeployment -ResourceGroupName $rgName -Name az400m05-vm0-deployment -TemplateFile .\az400m05-vm0.deployment.template.json -TemplateParameterFile .\az400m05-vm0.deployment.template.parameters.json -storageAccountName $storageAccountName -imageUrl $imageUrl
    ```

    > **备注**：等待预配过程完成。该过程需要约 5 分钟。 


#### 任务 2：配置 Azure DevOps 自托管代理

在此任务中，你需要将新预配的 Azure VM 配置为 Azure DevOps 自托管代理，并使用它来运行生成管道。

1.  在实验室计算机上，切换到显示 Azure 门户的 Web 浏览器窗口，在 Azure 门户中搜索并选择 **“虚拟机”** 资源类型，然后在 **“虚拟机”** 边栏选项卡上，单击 **“az400m05-vm0”**。
1.  在 **“az400m05-vm0”** 边栏选项卡上，单击 **“连接”**，在下拉列表中单击 **“RDP”**，然后在 **“az400m05-vm0\连接”** | 边栏选项卡上，单击 **“下载 RDP 文件”**，并使用远程桌面打开下载的 RDP 文件以连接到 **az400m05-vm0 Azure VM**。系统提示登录时，提供 **“Student”** 作为用户名，**Pa55w.rd1234** 作为密码。
1.  在 **az400m05-vm0** 远程桌面会话中，启动 Web 浏览器，导航到 [Azure DevOps 门户](https://dev.azure.com)，并使用与你的 Azure DevOps 组织相关联的 Microsoft 帐户登录。
1.  在 Azure DevOps 门户中，关闭 **“获取代理”** 面板，在 Azure DevOps 页的右上角，单击 **“用户设置”** 图标，在下拉菜单中，单击 **“个人访问令牌”**，然后在 **“个人访问令牌”** 窗格上，单击 **“+ 新建令牌”**。
1.  在 **“新建个人访问令牌”** 窗格上，单击 **“显示所有范围”** 链接，指定以下设置，然后单击 **“创建”**（将其他设置全部保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 名称 | **“配置代理池并了解管道样式”选项卡** |
    | 范围 | **代理池** |
    | 权限 | **读取和管理** |

1.  在 **“成功”** 窗格上，将个人访问令牌的值复制到剪贴板。

    > **备注**：确保复制令牌。关闭此窗格后，将无法再检索它。 

1.  在 **“成功”** 窗格中，单击 **“关闭”**。
1.  在 Azure DevOps 门户的“个人访问令牌”窗格上，单击左上角的 **“Azure DevOps”** 符号，然后单击左下角的 **“组织设置”** 标签。
1.  在 **“概述”** 窗格左侧的垂直菜单中，单击 **“管道”** 部分的 **“代理池”**。
1.  在 **“代理池”** 窗格上，单击右上角的 **“添加池”**。 
1.  在 **“添加代理池”** 窗格上，在 **“池类型”** 下拉列表中选择 **“自托管”**，在 **“名称”** 文本框中键入 **“az400m05l05a-pool”**，然后单击 **“创建”**。
1.  返回到 **“代理池”** 窗格，单击表示新创建的 **az400m05l05a-pool** 的条目。 
1.  在 **“az400m05l05a-pool”** 窗格的 **“作业”** 选项卡上，单击 **“代理”**标题。
1.  在 **“az400m05l05a-pool”** 窗格的 **“代理”** 选项卡上，单击 **“新建代理”** 按钮。
1.  在 **“获取代理”** 窗格上，确保选中 **“Windows”** 和 **“x64”** 选项卡，然后单击 **“下载”**，以将包含代理二进制文件的 zip 存档下载到你的用户配置文件中的本地 **“下载”** 文件夹。

    > **备注**： 如果你这时收到一条错误消息，指出当前系统设置阻止你下载该文件，请在 Internet Explorer 窗口中，单击右上角指示 **“设置”** 菜单标题的齿轮符号，在下拉菜单中选择 **“Internet 选项”**，在 **“Internet 选项”** 对话框中单击 **“高级”**，然后在 **“高级”** 选项卡上单击 **“重置”**，在 **“重置 Internet Explorer 设置”** 对话框中，再次单击 **“重置”**，然后单击 **“关闭”**，并重试下载。 

1.  在 **az400m05-vm0** 远程桌面会话中，启动 Windows PowerShell 作为管理员，并从 **“管理员: Windows PowerShell”** 控制台，运行以下命令以创建 **“C:\\agent”** 目录，并将已下载的存档内容提取到该目录：

    ```powershell
    New-Item -Type Directory -Path 'C:\agent'
    Set-Location -Path 'C:\agent'
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-2.179.0.zip", "$PWD")
    ```

1.  在 **az400m05-vm0** 远程桌面会话中，从 **“管理员:Windows PowerShell”** 控制台，运行以下命令以配置代理：

    ```powershell
    .\config.cmd
    ```

1.  出现提示时，指定以下设置的值：

    | 设置 | 值 |
    | ------- | ----- |
    | 输入服务器 URL | Azure DevOps 组织的 URL，格式为 **https://dev.azure.com/`<organization_name>`**，其中 `<organization_name>` 表示 Azure DevOps 组织的名称。 |
    | 输入身份验证类型（对于 PAT，按 enter） | **Enter** 键 | 
    | 输入个人访问令牌 | 你之前在此任务中记录的访问令牌 |
    | 输入代理池（对于默认内容，按 enter） | **az400m05l05a-pool** |
    | 输入代理名称 | **az400m05-vm0** |
    | 输入工作文件夹（对于 _work，按 enter） | **C:\agent** |
    | 输入“为每个步骤的任务执行 unzip”（按 enter 表示 N） | **Enter** |
    | 输入“将代理作为服务运行?”(Y/N)（按 enter 表示 N） | **Enter** |
    | 输入“配置自动登录并在启动时运行代理”(Y/N)（按 enter 表示 N） | **Enter** |

    > **备注**：可以将自托管代理作为服务或交互进程运行。你可能想使用交互模式启动，因为这可简化验证代理功能的操作。对于生产用途，你应考虑将代理作为服务运行或在启用自动登录的情况下将代理作为交互进程运行，因为这两种方式都会保存其运行状态，并确保在重启操作系统时代理会自动启动。

1.  在 **az400m05-vm0** 远程桌面会话中，从 **“管理员:Windows PowerShell”** 控制台，运行以下命令，以交互模式启动代理：

    ```powershell
    .\run.cmd
    ```

    > **备注**：验证代理是否报告 **“正在侦听作业”** 状态。

1.  切换到显示 Azure DevOps 门户的浏览器窗口，并关闭 **“获取代理”** 窗格。
1.  返回 **“az400m05l05a-pool”** 窗格的 **“代理”** 选项卡，请注意，列出了新配置的代理，其状态为 **“联机”**。
1.  在显示 Azure DevOps 门户的 Web 浏览器窗口中，单击左上角的 **“Azure DevOps”** 标签。
1.  在显示项目列表的浏览器窗口中，单击表示 **“配置代理池并了解管道样式”** 项目的磁贴。 
1.  在 **“配置代理池并了解管道样式”** 窗格上，在垂直导航窗格左侧，单击 **“管道”** 部分的 **“管道”**。 
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，选择 **“PartsUnlimited”**，然后在 **“PartsUnlimited”** 窗格上，选择 **“编辑”**。
1.  在 **“PartsUnlimited”** 编辑窗格上，在现有的基于 YAML 的管道中，将行 **6** `vmImage: vs2017-win2016`（指示目标代理池）替换为以下内容（指示新创建的自托管代理池）：

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```

1.  在 **“PartsUnlimited”** 编辑窗格上，单击窗格右上角的 **“保存”**，然后在 **“保存”** 窗格上，再次单击 **“保存”**。这将自动触发基于此管道的生成。 
1.  在 Azure DevOps 门户中，在垂直导航窗格左侧，单击 **“管道”** 部分的 **“管道”**。
1.  在 **“管道”** 窗格的 **“最近”** 选项卡上，单击 **“PartsUnlimited”** 条目，在 **“PartsUnlimited”** 窗格的 **“运行”** 选项卡上，选择最近的运行，在该运行的 **“摘要”** 窗格上，向下滚动至底部，在 **“作业”** 部分单击 **“阶段 1”**，并监视作业，直到作业成功完成。 

### 练习 3：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**：请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**：该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

#### 回顾

在本实验室中，你已了解如何将经典管道转换为基于 YAML 的管道，以及如何实现和使用自托管代理。
