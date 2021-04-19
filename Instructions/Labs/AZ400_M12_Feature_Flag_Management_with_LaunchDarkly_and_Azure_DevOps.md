---
lab:
    title: '实验室：使用 LaunchDarkly 和 Azure DevOps 进行功能标志管理'
    module: '模块 12：实现适当的部署模式'
---

# 实验室：使用 LaunchDarkly 和 Azure DevOps 进行功能标志管理
# 学生实验室手册

## 实验室概述

[**LaunchDarkly**](https://launchdarkly.com/) 是一个持续交付平台，能够以服务的形式提供功能标志。通过 LaunchDarkly，你能够将功能推出与代码部署分开，并大规模管理功能标志。将 LaunchDarkly 与 Azure DevOps 集成可最大程度减少与频繁发布相关联的潜在风险。若要进一步将发布与开发过程集成，可以将功能标志推出内容链接到 Azure DevOps 工作项。 

在本实验室中，你将了解如何利用 LaunchDarkly 在 Azure DevOps 中优化功能标志的管理。 

## 目标

完成本实验室后，你将能够：

- 在 LaunchDarkly 中创建功能标志
- 将 LaunchDarkly 与 Web 应用程序集成
- 自动在 Azure DevOps 发布管道中推出 LaunchDarkly 功能标志

## 实验室持续时间

-   预计用时：**60 分钟**

## 说明

### 准备工作

#### 登录实验室虚拟机

请确保已使用以下凭据登录到 Windows 10 虚拟机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge
-   可从 [Visual Studio 下载页面](https://visualstudio.microsoft.com/downloads/)获得 Visual Studio 2019 Community Edition。Visual Studio 2019 安装应包括“**ASP.NET 和 Web 开发**”、“**Azure 开发**”和“**NET Core 跨平台开发**”工作负载。它已预先安装在实验室计算机上。

#### 设置 LaunchDarkly 试用帐户

使用 Web 浏览器导航到 [LaunchDarkly 网站](https://launchdarkly.com/)并创建一个试用帐户。 

#### 设置 Azure DevOps 组织。

遵循[创建组织或项目集合](https://docs.microsoft.com/zh-cn/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明。

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板的预先配置的 Parts Unlimited 团队项目。 

#### 任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 **Parts Unlimited** 模板生成一个新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **备注**： 有关该站点的详细信息，请参阅 https://docs.microsoft.com/zh-cn/azure/devops/demo-gen。

1.  单击“**登录**”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1.  如果需要，在“**Azure DevOps 演示生成器**”页面上，单击“**接受**”以接受访问 Azure DevOps 订阅的权限请求。
1.  在“**新建项目**”页面上的页面的“**新建项目名称**”文本框中，键入“**LaunchDarkly**”，在“**选择组织**”下拉列表中选择你的 Azure DevOps 组织，然后单击“**选择模板**”。
1.  在模板列表中，单击工具栏中的“**DevOps 实验室**”，选择“**LaunchDarkly**”模板，然后单击“**选择模板**”。
1.  返回“**新建项目**”页面，如果系统提示你安装缺少的扩展，请选中“**LaunchDarkly Integration V2**”标签下方的复选框，然后单击“**创建项目**”

    > **备注**： 等待此过程完成。该过程需要约 2 分钟。如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在“**新建项目**”页面上，单击“**导航到项目**”。

### 练习 1：使用 LaunchDarkly 在 Azure DevOps 中配置功能标志

在本练习中，你将使用 LaunchDarkly 在 Azure DevOps 中配置功能标志。 

#### 任务 1：在 LaunchDarkly 中创建功能标志

在此任务中，你将在 LaunchDarkly 中创建功能标志

1.  在实验室计算机上，启动 Web 浏览器，导航到 [LaunchDarkly 网站](https://app.launchdarkly.com/)，然后使用你的帐户登录。浏览器会话将被重定向到“**默认项目**”门户，你可以在其中创建功能标志。 
1.  在 LaunchDarkly 门户的左侧垂直菜单栏中，单击“**功能标志**”。
1.  在“**功能标志**”窗格上，单击“**标志**”。
1.  在“**创建功能标志**”窗格上的“**名称**”文本框中，键入“**成员门户**”，然后单击“**保存标志**”按钮。

    > **备注**：你已经创建了一个名为“**成员门户**”的标志。假设你要使用此标志来确定 ASP.NET MVC Web 应用中“**成员门户**”功能的可见性。 

    > **备注**：若要将 LaunchDarkly 集成到应用程序中，你需要一个 SDK 密钥。 

1.  在 LaunchDarkly 门户的左侧垂直菜单栏中，单击“**帐户设置**”。 

    > **备注**：在“**帐户设置**”窗格上，你将找到两个预定义的环境： **生产**和**测试**。可以将生产环境 SDK 密钥用于该项目。 

1.  复制生产环境的 SDK 密钥并将其粘贴到记事本中。稍后将在本实验室用到它。

#### 任务 2：将 LaunchDarkly 集成到 Web 应用程序中

在此任务中，你需要将 LaunchDarkly 集成到 Web 应用程序中。

1.  在实验室计算机上，切换到显示 Azure DevOps 门户的 Web 浏览器窗口，其中 **LaunchDarkly** 项目处于打开状态，然后在项目窗格的左下角单击“**项目设置**”。
1.  在标题为“**项目设置**”的垂直菜单栏中，在“**Repos**”部分，选择“**存储库**”。
1.  在“**所有存储库**”窗格上，单击“**LaunchDarkly**”，然后在“**存储库设置**”窗格上，将“**提交提及链接**”和“**提交提及工作项解析**”设置设为“**开**”。
1.  在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“**Repos**”，然后在“**文件**”窗格上，单击“**克隆**”。
1.  在“**克隆存储库**”窗格上，单击指向下方的脱字符号，然后在下拉列表中单击“**Visual Studio**”。这将在“**Azure DevOps**”对话框处于打开状态的情况下自动启动 Visual Studio。 
1.  在 Visual Studio 窗口中的“**Azure DevOps**”对话框中，单击“**克隆**”。
1.  在 Visual Studio 窗口中，单击顶部“**视图**”菜单，然后在下拉菜单中单击“**Git 更改**”。 
1.  在 Visual Studio 窗口中“**Git 更改**”窗格顶部的“**master**”下拉列表中，单击向下箭头，在下拉对话框中单击“**远程**”，在远程分支列表中，单击“**origin/launch-darkly**”分支旁边的向下箭头，然后在下拉菜单中单击“**签出**”。 

    > **备注**： 等待，直到本地存储库中的文件更新完成。

1.  确保“**launch-darkly**”显示在 Visual Studio 窗口的右下角，切换到“**解决方案资源管理器**”窗口，然后单击 **PartsUnlimited.sln** 文件以打开解决方案。 

    > **备注**： 若要将 **LaunchDarkly** 与 .NET 应用程序集成，你需要在 **LaunchDarkly 客户**端上安装 NuGet 包。在当前项目中，为了易于使用，该包已经添加。

1.  在“**解决方案资源管理器**”窗格中，展开“**PartsUnlimitedWebsite**”节点，右键单击“**依赖项**”子节点，然后在右键菜单中选择“**管理 NuGet 包**”条目。
1.  在“**NuGet: PartsUnlimitedWebsite**”窗格中，请注意，已经安装了“**LaunchDarkly.Client**”。
1.  在 Visual Studio 界面的顶部菜单中，单击“**IIS Express**”，以在本地启动该应用程序。 
1.  验证该应用程序在本地浏览器会话中是否会成功启动，并且 Web 界面的右上角是否存在“**成员门户**”部分。

    > **备注**： 假设该“**成员门户**”模块是一个新功能并且你想要使用 **LaunchDarkly 功能**标志控制该功能。这样，当你在 LaunchDarkly 中启用该标志时，用过应该可以看到此功能。

1.  关闭显示 Web 应用程序界面的 Web 浏览器窗口。
1.  在 Visual Studio 窗口的“**解决方案资源管理器**”中，导航到 **PartsUnlimitedWebsite\\Controllers\\HomeController.cs** 并将其打开，将其内容替换为[更新后的 HomeController.cs 代码片段](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/HomeController.cs)中提供的代码，然后保存所做的更改。 
1.  在 Visual Studio 窗口的“**解决方案资源管理器**”中，导航到 **PartsUnlimitedWebsite\\Controllers\\AccountController.cs** 并将其打开，将其内容替换为[更新后的 AccountController.cs 代码片段](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/AccountController.cs)中提供的代码，然后保存所做的更改。 
1.  在 Visual Studio 窗口的“**解决方案资源管理器**”中，导航到 **PartsUnlimitedWebsite\\Views\Shared\\_Layout.chtml** 并将其打开，然后将第 55 行 `@await Html.PartialAsync(”_Login”)` 替换为以下代码。

    ```cshtml
    @if (ViewBag.togglevalue == true)
    {
      @await Html.PartialAsync("_Login")
    }
    ```

1.  切换到显示 **Home Controller.cs** 文件内容的选项卡，并将行 `static LdClient client = new LdClient(”__YourLaunchDarklySDKKey__”);` 中的占位符 `__YourLaunchDarklySDKKey__` 替换为在本练习的上一个任务中复制的 LaunchDarkly 帐户 SDK 密钥。 

    > **备注**： 这将使用特定于你的环境的 SDK 密钥创建一个 LdClient 对象。

1.  重复上一步，以更新 **AccountController.cs** 文件的代码。 

    > **备注**： 以上更改将导致 **HomeController** 从初始化静态 LaunchDarkly 客户端开始。查看 **MemberPortal** 的方法经过修改，可检查 LaunchDarkly 中的功能标志开关是处于“开”还是“关”状态。 **_Layout.cshtml** 页面检查开关值，并在标志打开的情况下呈现 MemberPortal 链接。 

1.  在 Visual Studio 窗口中，切换回“**HomeController.cs**”选项卡并查看其内容，注意第 57 - 74 行之间的代码：

    ```
        //LaunchDarkly start
        User user = LaunchDarkly.Client.User.WithKey("administrator@test.com");
        bool value = client.BoolVariation("member-portal", user, false);
        if (value)
        {
          ViewBag.Message = "Your application description page.";
          ViewData["togglevalue"] = value;
          return View(viewModel);
        }
        else
        {
          return View(viewModel);
        }
        // return View(viewModel);

    }
    //LaunchDarkly End
    ```

    > **备注**： 当请求功能标志时，你需要传递一个用户对象。因此，我们在一开始就对用户对象进行初始化。这将用于检查 LaunchDarkly 中是否存在具有指定密钥的用户。在此示例中，我们已对用户值进行了硬编码。在实际情况下，可以通过标识已登录的用户或执行数据库查找来检索此值。

    > **备注**： 然后，我们将调用 **BoolVariation** 方法来检查 LaunchDarkly 中的功能标志值。如果该标志为 true，则 **[ViewData[”togglevalue”]** 将设置为 true，你可以在 **_Layout.chtml** 中使用该值查看“成员门户”模块。如果它为 false，则不会显示“会员门户”模块。

    > **备注**：同样，在 **AccountController.cs** 中，我们向 **Login()** 方法添加了 LaunchDarkly 代码，该代码负责在单击“**成员门户**”图标后显示“登录”页面。如果 LaunchDarkly 中的标志为 false，则“登录”页面将返回 HttpNotFound 错误。

1.  在 Visual Studio 窗口中，单击“**全部保存**”，然后单击“**IIS Express**”以在本地启动应用程序。 
1.  验证“**成员门户**”链接是否不再出现在显示 Parts Unlimited 网站的 Web 浏览器中，因为此时 **MemberPortal** 标志处于关闭状态。

    > **备注**： 这里使用 LaunchDarkly 实现功能标志控制。现在可以手动从 LaunchDarkly 门户启用开关。但在本实验室中，我们将使用 LaunchDarkly 扩展通过 Azure DevOps 发布管道对其进行配置。为了在发布过程中加入功能标志，我们会将此更改与 Azure DevOps 工作项关联。 

1.  关闭显示 Web 应用程序界面的 Web 浏览器窗口。
1.  在 Visual Studio 窗口的“**团队资源管理器**”窗格中，单击工具栏中的“**主页**”图标，然后在“**主页**”上单击“**工作项**”。
1.  在“**团队资源管理器**”窗格中的“**工作项**”页面上，右键单击“**我的查询**”，然后在右键菜单中单击“**新建查询**”。这将自动在 Visual Studio 窗口的中心窗格中打开“**新建查询**”选项卡。
1.  在“**新建查询**”窗格中，接受默认设置，然后单击“**运行**”。这将返回与此分支关联的单个工作项。
1.  在查询结果中，记下工作项 ID，然后单击代表工作项的条目。这将自动在 Visual Studio 窗口的中心窗格中打开另一个选项卡，该选项卡代表与该工作项对应的用户故事。
1.  在“用户故事”选项卡的左上角，单击“**未分配**”，键入你的 Azure DevOps 用户名并按 **Enter** 键，将工作项分配给你自己，然后单击“**保存工作项**”以保存所做的更改。
1.  在 Visual Studio 窗口中，切换到“**Git 更改**”窗格，在“**输入消息**”文本框中，键入“**Integated LaunchDarkly #<work\_item\_ID>**”，其中 **<work\_item\_ID>** 代表你先前记下的 ID，单击“**全部提交**”按钮旁边的向下箭头，然后在下拉列表中单击“**全部提交并推送**”。 

#### 任务 3：发布过程中自动推出 LaunchDarkly 功能标志

在此任务中，你将在 Azure DevOps 发布过程中配置自动推出 LaunchDarkly 功能标志

> **备注**： 我们需要一个 LaunchDarkly 访问令牌才能与 Azure DevOps 服务集成。 

1.  若要检索 LaunchDarkly 访问令牌，请切换回显示 LaunchDarkly 门户的浏览器窗口，在左侧的垂直菜单中，单击“**帐户设置**”，在“**帐户设置**”窗格上，单击“**授权**”选项卡，然后在“**访问令牌**”部分，单击“**创建令牌**”。
1.  在“**创建访问令牌**”窗格上的“**名称**”文本框中，键入“**成员门户**”，在“**角色**”下拉列表中，选择“**写入者**”，然后单击“**保存令牌**”。
1.  返回到“**帐户设置**”窗格的“**授权**”选项卡，复制访问令牌并将其粘贴到记事本中。 

    > **备注**： 确保复制令牌。离开当前页面后，你将无法检索它。

1.  切换回显示 Azure DevOps 门户的浏览器窗口，然后在“**LaunchDarkly**”项目窗格上，单击“**项目设置**”。 
1.  在“**项目详细信息**”窗格上的“**管道**”部分，单击“**服务连接**”，再单击“**创建服务连接**”。
1.  在“**新建服务连接**”窗格上，单击“**LaunchDarkly**”，然后单击“**下一步**”。
1.  在“**新建 LaunchDarkly 服务连接**”窗格上的“**访问令牌**”文本框中，粘贴之前在此任务中检索到的访问令牌，在“服务连接名称”中，键入“**az-400 m12l01 LaunchDarkly**”，然后单击“**保存**”。
1.  在 Azure DevOps 门户的左侧垂直菜单中，单击“**版块**”，然后在“**工作项**”窗格上，单击在上一个任务中分配给你自己的工作项。
1.  在“**使用 LaunchDarkly 实现 FeatureFlag 管理**”工作项窗格上，选择右上角的“**启动 Darkly**”选项卡。 
1.  在“**LauchDarkly**”选项卡上的“**选择功能标志**”部分，选择“**成员门户**”功能标志。
1.  在 Azure DevOps 门户左侧的垂直菜单栏中，单击“**管道**”，然后在“**管道**”部分，选择“**发布**”。 
1.  在“**管道/发布**”窗格上，选择“**LaunchDarkly_CD**”管道，然后单击“**编辑**”。
1.  在“**所有管道/LaunchDarkly_CD**”窗格上，单击“**阶段 1**”阶段的“**1 个作业，3 个任务**”链接以查看管道中的任务。
1.  确认阶段包含以下任务：

    - **Azure 资源组部署**：此任务使用 ARM 模板为 PartsUnlimited 网站部署 Azure 应用服务。
    - **LaunchDarkly 推出**：此任务将打开 LaunchDarkly 订阅中的功能标志。
    - **Azure 应用服务部署**：此任务将 PartsUnlimited Web 应用部署到在第一个任务中创建的 Azure 应用服务。

1.  在任务列表中，选择“**Azure 资源组部署**”任务，在“**Azure 订阅**”下拉列表中，单击指向下方的脱字符，在条目列表中，选择要在本实验室中使用的 Azure 订阅，然后单击“**授权**”以配置 Azure 服务连接。如果系统提示使用具有 Azure 订阅中的所有者角色和与 Azure 订阅关联的 Azure AD 租户中的全局管理员角色的帐户登录，
1.  在任务列表中，选择“**LaunchDarkly 推出**”任务，然后在“**帐户**”下拉列表中选择代表你先前在此任务中创建的 LaunchDarkly 服务连接的条目。

    > **备注**： 验证“**标志状态**”是否设置为“**开**”。此设置控制发布过程中标志的状态。

1.  在任务列表中，选择“**Azure 应用服务部署**”任务，在“**Azure 订阅**”下拉列表中，选择先前创建的 Azure 订阅服务连接。 
1.  在 Azure DevOps 门户的 Azure DevOps 页面右上角，单击“**用户设置**”图标，在下拉菜单中，单击“**个人访问令牌**”，然后在“**个人访问令牌**”窗格上，单击“**新建令牌**”。
1.  在“**创建新的个人访问令牌**”窗格上，指定以下设置，然后单击“**创建**”（所有其他设置保留默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 名称 | **LaunchDarkly 和 Azure DevOps 实验室** |
    | 范围 | **完全访问权限** |

1.  在“**成功**”窗格上，将个人访问令牌的值复制到剪贴板。

    > **备注**： 确保复制令牌。关闭此窗格后，将无法再检索它。 

1.  在“**成功**”窗格中，单击“**关闭**”。
1.  导航回“**所有管道/LaunchDarkly_CD**”窗格，然后单击“**变量**”选项卡。 
1.  在变量列表中，将 **launchdarkly-pat** 变量的值设置为新生成的个人访问令牌。
1.  在变量列表中，将 **launchdarkly-project-name** 变量的值设置为 **LaunchDarkly**。

    > **备注**： 这将完成发布管道的配置。 

1.  在 Azure DevOps 门户左侧垂直菜单栏中的“**管道**”部分，选择“**管道**”。
1.  在“**管道**”窗格上，单击代表 **LaunchDarkly-CI** 生成管道的条目，在“**LaunchDarkly-CI**”窗格上，单击“**运行管道**”，然后在“**运行管道**”窗格上，单击“**运行**”。

    > **备注**： 此 CI 管道编译 .Net Core 项目。生成成功完成后，将触发发布以在 LaunchDarkly 中部署应用和推出功能标志。

1.  在生成管道运行窗格的“**作业**”部分，单击“**代理作业 1**”条目，并监视生成过程的进度。
1.  生成完成后，在 Azure DevOps 门户左侧垂直菜单栏中的“**管道**”部分，选择“**发布**”。
1.  在“**管道/发布**”窗格上的“**LaunchDarkly_CD**”窗格上，单击“**Release-1**”条目，并监视部署过程的进度。

    > **备注**： 部署成功完成后，你应该能够在 LaunchDarkly 仪表板中看到 **MemberPortal** 功能标志处于打开状态。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用具有在本实验室中使用的 Azure 订阅中的所有者角色的用户帐户登录。
1.  在 Azure 门户中，搜索并选择“**应用服务**”资源，然后从“**应用服务**”边栏选项卡，导航到通过 Azure DevOps 发布管道在其中部署应用程序的 Web 应用。
1.  在 Web 应用边栏选项卡上，单击“**浏览**”。这将打开另一个 Web 浏览器标签页，其中显示了新部署的 Web 应用程序。
1.  在“Parts Unlimited”网页上，验证是否启用“**成员门户**”功能。

> **备注**： LaunchDarkly 提供了许多其他功能，包括：
    
- **用户目标确定**：借助 LaunchDarkly 目标确定功能，你可以为单个用户或用户组启用或关闭功能。在更广泛地推出功能之前，你可以使用它推出用于内部测试、专用测试版或可用性测试的功能。 
- **自定义目标确定规则**：除了面向单个用户外，LaunchDarkly 还允许通过构建自定义规则来面向用户群。换句话说，可以基于指定的任何属性创建面向用户的自定义规则。 
- **用于管理开发过程的项目和环境**：[项目](https://docs.launchdarkly.com/docs/projects)允许在一个 LaunchDarkly 帐户下管理多个不同的软件项目。[环境](https://docs.launchdarkly.com/docs/environments)允许在整个开发生命周期（从本地开发到 QA、暂存和生产）中管理功能标志。 

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**： 请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure 门户删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，导航到你之前在本实验室中部署的应用服务实例。 
1.  在“应用服务实例”边栏选项卡上，单击表示包含应用服务实例的资源组名称的链接。
1.  在包含应用服务实例的资源组的边栏选项卡上，单击“**删除资源组**”，当出现提示时，提供资源组的名称，然后单击“**删除**”。

## 回顾

在本实验室中，你学习了如何利用 LaunchDarkly 在 Azure DevOps 中优化功能标志的管理。 
