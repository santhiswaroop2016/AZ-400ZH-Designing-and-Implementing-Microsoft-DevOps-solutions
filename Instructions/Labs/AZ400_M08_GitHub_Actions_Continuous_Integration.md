---
lab:
    title: '实验室：GitHub Actions 持续集成'
    module: '模块 8：实现与 GitHub Actions 的持续集成'
---

# 实验室：GitHub Actions 持续集成
# 学生实验室手册

## 实验室概述

GitHub Actions 简化了将持续集成 (CI) 合并到 GitHub 存储库的操作。在本实验室中，你将设置两个可自动化生成过程的 GitHub 工作流。

> **备注**： 在本实验室中，你将与 **github-learning-lab** 机器人交互，它将指导你完成各个实验室任务。

## 目标

完成本实验室后，你将能够：

- 描述 GitHub Actions 在持续集成中的重要性
- 使用和自定义模板化工作流
- 创建与你的团队需求和行为匹配的 CI 工作流
- 使用存储库的源代码并在工作流的作业中生成项目
- 使用 GitHub Actions 实现单元测试框架
- 创建运行测试和生成测试报告的工作流
- 设置矩阵生成，以创建适用于多个目标平台的生成项目
- 保存和访问存储库的生成项目
- 选择虚拟环境以满足应用程序的 CI 需求

> **备注**：在本实验室中，许多任务将与生成相关。构建这些生成有时会花更长的时间，最多可能需要几分钟。请确保等待生成完成后，再继续执行下一步。

## 实验室持续时间

-   预计用时：**150 分钟**

## 说明

### 准备工作

#### 登录实验室虚拟机

确保已使用以下凭据登录到 Windows 10 计算机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### 设置 GitHub 帐户

如果尚无可用于本实验室的 GitHub 帐户，请按照[注册新 GitHub 帐户](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account)中的说明操作。

### 练习 1：实现与 GitHub Actions 的持续集成

在此演练中，你将实现与 GitHub Actions 的持续集成

#### 任务 1：将模板化的工作流添加到 GitHub 存储库

在此任务中，你将按照以下步骤创建具有模板化工作流的拉取请求：

- 导航到“操作”选项卡。
- 选择模板 Node.js 工作流。
- 将工作流提交到新分支。
- 创建名为 CI for Node 的拉取请求。

1.  在实验室计算机上启动 Web 浏览器，导航到 [GitHub Actions：持续集成实验室](https://lab.github.com/githubtraining/github-actions:-continuous-integration)的启动页面，如果尚未登录 GitHub，请单击右上角的 **“登录”**。
1.  在 **“GitHub 学习实验室”** 页面上，单击 **“开始学习 GitHub 学习实验室”**。 
1.  返回 [GitHub Actions: 持续集成实验室](https://lab.github.com/githubtraining/github-actions:-continuous-integration)的启动页面，单击 **“开始学习免费课程”**。
1.  在弹出窗口中，请注意，**GitHub 学习实验室**将在你的 GitHub 帐户中创建名为 **github-actions-for-ci** 的公共存储库，然后单击 **“启动 GitHub Actions: 持续集成”**。
1.  返回 **GitHub Actions: 持续集成”** 页面的启动页面，在 **“课程步骤”** 列表中，单击第一步 **“使用模板化工作流”** 旁边的 **“开始”**。你将被自动重定向到 **github-actions-for-ci** 存储库的 **“问题”** 选项卡。
1.  在 **github-actions-for-ci** 存储库的 **“问题”** 选项卡中，在 **“存在 bug”** 问题页面上查看 **“欢迎”** 部分。 

    > **备注**：根据 **“欢迎”** 部分的内容，存储库中的某个位置存在 bug。你可使用持续集成 (CI) 做法来设置自动化测试，以便更轻松地发现、诊断和最大程度减少此类情况。代码库是使用 Node.js 编写的。借助 GitHub Actions，可将一些模板化工作流用于常用语言和框架（例如 Node.js）。你将创建带有模板化工作流的拉取请求。

1.  在 **“存在 bug”** 页面上，单击 **“操作”** 顶部菜单选项卡标头。 
1.  在 **github-actions-for-ci** 存储库的 **“操作”** 选项卡面上，在 **“GitHub Actions 入门”** 页面上的 **“Node.js”** 窗格中单击 **“设置此工作流”**。
1.  在 **“github-actions-for-ci/.github/workflows/node.js.yml”** 页面上，单击 **“开始提交”**。
1.  在 **“提交新文件”** 窗格上，接受将为此提交新建分支并发出拉取请求的默认设置，然后单击 **“提交新文件”**。
1.  在 **“打开拉取请求”** > **“创建 node.js.yml”** 页面中，将拉取请求名称 **“创建 node.js.yml”** 重命名为 **“CI for Node”**，然后单击 **“打开拉取请求”**。

> **备注**：将模板化工作流添加到分支后，GitHub Actions 即可为存储库启动 CI。 

#### 任务 2：运行模板化工作流

在此任务中，你将跟踪模板化工作流的执行并查看结果

1.  在 **github-actions-for-ci GitHub** 存储库的 **“CI for Node #2”** 页面的 **“拉取请求”** 选项卡上，在拉取请求的 **“对话”** 选项卡上查看其预期更改，该更改表示 **github/workflows/node.js.yml** 文件的内容。
1.  查看各更改时，请单击相应的 **“解决对话”** 按钮。

    > **备注**：**github/workflows/node.js.yml** 文件包含以下组件：

    - 工作流：工作流是自动化的一个单元，它包含以下方面的定义：自动化的触发因素、自动化期间应考虑的环境因素或其他方面，以及触发后应发生的操作。
    - 作业：作业是工作流的一部分，由一个或多个步骤组成。在示例工作流中，模板定义了构成生成作业的步骤。
    - 步骤：一个步骤表示自动化的一个部分。可以将一个步骤定义为一个 GitHub Action。
    - 操作：GitHub Action 是自动化的一部分，是以与工作流兼容的方式编写的。你可以使用直接从 GitHub 或从开源社区获取的内置操作，也可以创建自定义操作。例如，**actions/checkout@v2** 用于确保运行生成的虚拟机具有代码库的副本。签出的代码用于运行测试。**action/setup-node@v1** 操作用于设置适当版本的 Node.js，因为我们将对多个版本执行测试。

    > **备注**：**github/workflows/node.js.yml** 文件中的 **“on:”** 字段可向 GitHub Actions 告知运行时间。在这种情况下，只要有推送，我们就会运行工作流。

    > **备注**：**github/workflows/node.js.yml** 文件中的 **jobs:** 块定义了 Actions 工作流的核心组件。工作流由作业构成，我们的模板工作流使用标识符生成来定义作业。每个作业还需要在特定的主机上运行，该主机由 **“runs-on:”** 字段的值指定。此模板工作流使用最新版本的 Ubuntu 运行生成作业。

    > **备注**：除了运行预生成操作之外，工作流还可以执行命令，就像你直接访问运行生成的虚拟机时一样。通过使用模板工作流的 **“run:”** 字段，你可以运行与项目相关的任意命令，例如用于安装依赖项的 **npm install** 或用于运行所选测试框架的 **npm test**。

    ```yaml
    # 此工作流将进行节点依赖项的全新安装、生成源代码，并在不同版本的节点上运行测试
    # 有关详细信息，请参阅：https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

    name: Node.js CI

    on:
      push:
        branches: [ master ]
      pull_request:
        branches: [ master ]

    jobs:
      build:

        runs-on: ubuntu-latest

        strategy:
          matrix:
            node-version: [10.x, 12.x, 14.x]

        steps:
        - uses: actions/checkout@v2
        - name: Use Node.js ${{ matrix.node-version }}
          uses: actions/setup-node@v1
          with:
            node-version: ${{ matrix.node-version }}
        - run: npm ci
        - run: npm run build --if-present
        - run: npm test
    ```

> **备注**：请忽略工作流将失败这一事实。这在预料之中。要确定失败的原因，可以查看存储库的 **“操作”** 选项卡中的内容，也可以单击与拉取请求的 **“对话”** 选项卡上列出的失败相对应的 **“详细信息”** 链接。

> **备注**：在我们的示例中，错误是 **npm test** 命令导致的。**npm test** 命令用于查找测试框架。本实验室中将使用 Jest。Jest 需要在名为 **\_\_test\_\_** 的目录中进行单元测试。问题在于，此分支上不存在 **\_\_test\_\_** 目录。

#### 任务 3：向 GitHub 工作流添加 jest 测试

在此任务中，你将按照以下步骤向 GitHub 工作流添加基于 Jest 的测试，以解决在上一任务中确定的失败原因：

- 导航到名为 **“添加 Jest 测试”** 的开放拉取请求。
- 合并拉取请求

1.  在 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 页面上，单击 **“拉取请求”** 选项卡。
1.  在拉取请求列表中，单击 **“添加 jest 测试”**。
1.  在 **“添加 jest 测试”** 页面上，查看拉取请求的 **“对话”** 选项卡的内容。

    > **备注**：此拉取请求引入了 Jest，这是一种常用的 JavaScript 测试框架。我们将使用它，并了解如何使用它进行持续集成。

1.  在 **“添加 jest 测试”** 页面上，在拉取请求的 **“对话”** 选项卡中单击 **“合并拉取请求”**，然后单击 **“确认合并”**。
1.  在 **“添加 jest 测试”** 页面上的拉取请求的 **“对话”** 选项卡中，在 **github-learning-lab** 机器人的注释中单击 **“下一步”** 链接。这会将你重定向回 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 拉取请求的 **“对话”** 选项卡。

#### 任务 4：查看 Actions 日志并确定失败的测试

在此任务中，你将查看 Actions 日志并确定失败的测试。 

> **备注**：现已正确配置测试框架，生成过程应该会自动调用它。你可查看相应日志以确定结果并建议下一个步骤。与之前一样，可以通过存储库的“操作”选项卡，或通过与拉取请求的“对话”选项卡上列出的失败相对应的 **“详细信息”** 链接来访问日志。

要完成此任务，需按顺序执行以下步骤：

- 导航到工作流日志
- 确定失败的测试的名称
- 向对话添加注释，并在其中包含失败的测试的名称

1.  返回到 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 拉取请求的 **“对话”** 选项卡，向下滚动到 **github-learning-lab** 机器人的 **“等待测试”** 注释并查看其内容。
1.  向下滚动到 **“对话”** 选项卡上的注释列表，然后单击各失败的检查旁边的 **“详细信息”** 链接。这会自动重定向到 **“CI for Node #2”** 的 **“检查”** 选项卡。
1.  在 **“CI for Node #2”** 的 **“检查”** 选项卡上，查看每个生成日志，并验证它们在 **“运行 npm 测试”** 期间是否全都失败。
1.  探索 **“运行 npm 测试”** 阶段，然后在 **“游戏”** 部分中，确定带 **X** 标记（表示失败）的各测试的名称。

    ```
    FAIL __test__/game.test.js
      App
        ✕ Contains the compiled JavaScript (8 ms)
      Game
        Game
          ✕ Initializes with two players (3 ms)
          ✓ Initializes with an empty board (1 ms)
          ✕ Starts the game with a random player
        turn
          ✓ Inserts an 'X' into the top center
          ✓ Inserts an 'X' into the top left
        nextPlayer
          ✕ Sets the current player to be whoever it is not (1 ms)
        hasWinner
          ✓ Wins if any row is filled (1 ms)
          ✓ Wins if any column is filled
          ✓ Wins if down-left diagonal is filled (1 ms)
          ✓ Wins if up-right diagonal is filled
    ```

1.  切换回 **“对话”** 选项卡，向下滚动到注释列表的最底部，在最新注释的 **“写入”** 选项卡上，将 **“添加注释”** 替换为在上一步中确定的以下名称，然后单击 **“注释”**。

    ```
    Initializes with two players
    Starts the game with a random player
    Sets the current player to be whoever it is not
    ```

> **备注**：这将在 **“对话”** 选项卡上显示机器人的另一组注释，首先显示的是注释 **“读取失败的日志”**。

#### 任务 5：修复失败的测试

在此任务中，你将修改导致测试失败的文件，以纠正问题。

> **备注**：一个失败的测试是：初始化两个玩家。如果深入了解日志的更多详细信息，你可能会注意到，单元测试要求将两个玩家分别命名为 Salem 和 Nate，但由于某种未知的原因 Nate 被替换为了 Bananas：

    ```
      ● Game › Game › Starts the game with a random player

        expect(received).toBe(expected) // Object.is equality

        Expected: "Nate"
        Received: "Bananas"

          39 | 
          40 |       Math.random = () => 0.6
        > 41 |       expect(new Game(p1, p2).player).toBe('Nate')
             |                                       ^
          42 |     })
          43 |   })
          44 | 
    ```

> **备注**：通常的做法是为测试文件提供与要测试的代码文件相同的名称，然后直接添加 .test.js 扩展名。可以假设，game.test.js 的测试结果是由 game.js 中的问题导致的。

1.  在 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 拉取请求的 **“对话”** 选项卡上，找到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  要确定注释中引用的代码变更，请单击 **“查看更改”** 按钮，向下滚动到表示 **\_\_test\_\_/game.test.js** 文件的部分，然后查看建议的更改。

    ```yaml
    Suggested change 
        this.p2 = 'Bananas'
        this.p2 = p2
    ```

1.  导航回 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 拉取请求的 **“对话”** 选项卡，向下滚动到 **github-learning-lab** 机器人的最新注释，单击 **“注释建议”**，并在弹出窗格中单击 **“注释变更”**。

    > **备注**：提交更改后，测试将再次运行并成功完成。 

1.  在 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 拉取请求的 **“对话”** 选项卡上，向下滚动到 **“批准的更改”** 响应，依次单击 **“合并拉取请求”** 和 **“确认合并”**。 
1.  在 **github-actions-for-ci** GitHub 存储库的 **“CI for Node #2”** 拉取请求的 **“对话”** 选项卡上，在 **github-learning-lab** 机器人的最新注释中，单击 **“下一步”** 链接。这会将你重定向到 **“问题”** 选项卡，其中显示了 **“整个团队的工作流 #4”** 页面。

#### 任务 6：查看后续步骤

在此任务中，你将查看后续步骤，通过这些步骤，你可以共享你的第一个工作流，以便整个团队都可使用它。

> **备注**：你已了解如何设置 CI，现在让我们来尝试更实际的用例。你的团队有一个自定义工作流，它超越了我们迄今为止使用的模板的范畴。我们需要以下功能：

- 针对多个目标进行测试的功能，以便能够验证操作系统和 Node.js 版本的不同组合
- 专用测试作业，以便我们将生成与测试详细信息区分开
- 对生成项目的访问权限，以便我们能够将这些项目部署到目标环境
- 分支保护，以确保无法删除或意外破坏主分支
- 必要的审查，以便团队成员可以复核任意拉取请求
- 显式批准，这样我们就可以快速合并，并可能自动完成合并和部署

#### 任务 7：创建自定义 GitHub Actions 工作流

在此任务中，你将创建自定义 GitHub Actions 工作流。

要实现此功能，需按顺序执行以下步骤：

- 在新分支中编辑现有工作流文件
- 在工作流文件中，将 12.x 和 14.x 版 Node 作为目标版本
- 打开名为 **“改进 CI”** 的新拉取请求，其中包括你所做的更改

1.  在 **github-actions-for-ci** 存储库的“问题”选项卡面上（其中显示了 **“整个团队的工作流 #4”** 页面），向下滚动到标记为 **“步骤 7: 创建自定义 GitHub Actions 工作流”** 的部分，在 **“活动: 使用新的生成目标编辑现有工作流”** 列表中,单击 **“现有工作流”** 链接。这会将你重定向到 **github-actions-for-ci** 存储库的 **“代码”** 选项卡面上 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的 **“编辑文件”** 视图。
1.  在 **github-actions-for-ci/.github/workflows/node.js.yml** 文件中，将 `node-version: [10.x, 12.x, 14.x]` 替换为 `node-version: [12.x, 14.x]`
1.  在 **github-actions-for-ci** 存储库的“代码”选项卡面上，单击右上角的 **“开始提交”**。
1.  在 **“提交更改”** 窗格上，接受将为此提交新建分支并发出拉取请求的默认设置，然后单击 **“提交更改”**。
1.  在 **“打开拉取请求”** 页面中，将拉取请求的名称 **“更新 node.js.yml”** 重命名为 **“改进 CI”**，然后单击 **“打开拉取请求”**。这会自动重定向到 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“创建 CI #5”** 页面。

> **备注**：通过以特定版本的 Node 为目标，你配置了生成矩阵，该矩阵可用于跨多个操作系统、平台和语言进行测试。有关此主题的详细信息，请参阅 [GitHub 文档](https://help.github.com/en/articles/configuring-a-workflow#configuring-a-build-matrix)。

#### 任务 8：以 Windows 环境为目标

在此任务中，你将编辑工作流文件以针对 Windows 环境进行生成

> **备注**：由于我们要为将应用部署到 Windows 环境提供支持，因此需要将 Windows 添加到矩阵生成配置。

要实现此功能，需按顺序执行以下步骤：

- 将 os 字段添加到工作流的 **strategy.matrix** 部分
- 将 **ubuntu-latest** 和 **windows-2016** 添加到目标操作系统列表
- 将工作流更改提交到现有分支

    ```yaml
    node-version: [10.x, 14.x]
    os: [ubuntu-latest, windows-2016]
    node-version: [12.x, 14.x]
    ```

1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面上，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **“活动: 编辑工作流文件以针对 Windows 环境进行生成”** 部分中，单击步骤 1 中的 **github/workflows/nodejs.yml** 链接。  这会将你重定向到 github-actions-for-ci 存储库的“代码”选项卡面上 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的 **“编辑文件”** 视图。
1.  在 **github-actions-for-ci/.github/workflows/node.js.yml** 文件中，查看矩阵部分并确定应进行的更改。 
1.  导航回 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡面上 **“改进 CI #5”** 页面的 **“活动: 编辑工作流文件以针对 Windows 环境进行生成”** 部分，单击 **“提交建议”**，然后在弹出窗格中单击 **“提交更改”**。
1.  这将使用 **github-learning-lab** 机器人的最新注释（标记为 **“新建作业”**），更新 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡面上 **“改进 CI #5”** 页面的内容。

> **备注**：通过查看日志，你可发现目前有 4 个生成。这是因为对于两个操作系统中的每一个，我们都会针对两个版本运行测试。

#### 任务 9：配置多个作业

在此任务中，你将编辑工作流文件以分离生成和测试作业

> **备注**：我们现在开始创建专用测试作业。这样，我们可以将工作流的生成和测试功能分离为多个作业，这些作业将在触发工作流时运行。

1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面上，在 **github-learning-lab** 机器人的最新注释（标记为 **“新建作业”**）的 **“步骤 9: 使用多个作业”** 部分中，单击步骤 1 中的 **“你的工作流文件”** 链接。 这会将浏览器重定向到 **github-actions-for-ci** 存储库的 **“代码”** 选项卡面上 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的 **“编辑文件”** 视图。
1.  在生成作业部分，添加表示现有工作流的以下内容：

    ```yaml
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: npm install and build webpack
          run: |
            npm install
            npm run build
    ```

1.  在 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的最底部，添加一个表示名为 **“测试”** 的作业（确保其缩进与生成作业条目匹配）的新条目，该条目的内容如下：

    ```yaml
    test:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          os: [ubuntu-latest, windows-2016]
          node-version: [12.x, 14.x]
      steps:
      - uses: actions/checkout@v2
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: npm install, and test
        run: |
          npm install
          npm test
        env:
          CI: true
    ```

1.  这将生成以下内容：

    ```yaml
    name: Node CI

    on: [push]

    jobs:
      build:

        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v2
          - name: npm install and build webpack
            run: |
              npm install
              npm run build

      test:

        runs-on: ubuntu-latest

        strategy:
          matrix:
            os: [ubuntu-latest, windows-2016]
            node-version: [12.x, 14.x]

        steps:
        - uses: actions/checkout@v2
        - name: Use Node.js ${{ matrix.node-version }}
          uses: actions/setup-node@v1
          with:
            node-version: ${{ matrix.node-version }}
        - name: npm install, and test
          run: |
            npm install
            npm test
          env:
            CI: true
    ```

1.  在 **github-actions-for-ci** 存储库的 **“代码”** 选项卡（显示正在编辑的 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的内容）上，单击右上角的 **“开始提交”**。
1.  在 **“提交更改”** 窗格上，接受默认设置（使用此设置时，更改将直接提交到当前分支），然后单击 **“提交更改”**。

> **备注**：提交之后，工作流将再次运行。 

#### 任务 10：运行多个作业
 
在此任务中，你只需等待工作流中多个作业的结果。

> **备注**：在此任务中，无需进行任何操作。 

#### 任务 11：上传作业的生成项目

在此任务中，你将在工作流文件中使用上传操作，以保存作业的生成项目

> **备注**：你可能会注意到虽然已成功完成生成，但每个测试作业都失败了。这是因为所有作业均在虚拟环境的新实例中执行，导致在生成中创建的生成项目不可用于测试作业。这是虚拟环境设计的固有缺陷。要解决这个问题，我们可以使用内置的项目存储来保存从一个作业中创建的项目，以便在同一工作流的其他作业中使用该项目。通过使用项目，可在作业完成后保存数据，并与同一工作流中的其他作业共享该数据。项目是指在工作流运行期间生成的文件或文件集合。

> **备注**：要将项目上传到项目存储，我们可以使用通过 GitHub: actions/upload-artifacts 生成的操作。

1.  导航回 **github-actions-for-ci GitHub** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面，在 **“对话”** 选项卡上，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面上，在 **github-learning-lab** 机器人的最新注释（标记为 **“新建作业”**）的 **“步骤 11: 上传作业的生成项目”** 部分中，单击步骤 1 中的 **“工作流文件”** 链接。这会将浏览器重定向到 **github-actions-for-ci** 存储库的“代码”选项卡面上 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的 **“编辑文件”** 视图。
1.  在生成作业部分，更新生成作业，在其中加入 **upload-artifacts** 操作，使其与以下内容匹配：

    ```yaml
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2
          - name: npm install and build webpack
            run: |
              npm install
              npm run build
          - uses: actions/upload-artifact@master
            with:
              name: webpack artifacts
              path: public/
    ```

1.  在 **github-actions-for-ci** 存储库的“代码”选项卡（显示正在编辑的 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的内容）上，单击右上角的 **“开始提交”**。
1.  在 **“提交更改”** 窗格上，接受默认设置（使用此设置时，更改将直接提交到当前分支），然后单击 **“提交更改”**。

> **备注**：提交之后，工作流将再次运行。 

#### 任务 12：下载作业的生成项目

在此任务中，你将在工作流文件中进行下载操作，以访问先前作业的生成项目

> **备注**：生成项目现在将被上传到项目存储，但如果你跟踪工作流，就会发现测试作业仍然失败。这是由于两个主要原因：

- 除非明确将作业配置为按顺序运行，否则其将并行运行。
- 每个作业都在其自己的虚拟环境中运行，因此尽管已将项目推送到了存储，但测试作业仍需要检索它们。

> **备注**：为了解决这个问题，你需要在生成完成后运行测试，以便项目可用。此外，为了通过上传操作来将项目复制到指定的项目存储，我们将通过另一个 GitHub 内置操作 **actions/download-artifact** 来下载这些项目。

1.  导航回 **github-actions-for-ci** GitHub 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面，在 **“对话”** 选项卡上，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面上，在 **github-learning-lab** 机器人的最新注释（标记为 **“新建作业”**）的 **“步骤 12: 下载作业的生成项目”** 部分中，单击步骤 1 中的 **“工作流文件”** 链接。这会将浏览器重定向到 **github-actions-for-ci** 存储库的 **“代码”** 选项卡面上 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的 **“编辑文件”** 视图。
1. 在测试作业部分，将测试作业更新为仅在生成作业完成后才运行，方法是添加 **“needs: build”** 组件，使作业与以下内容匹配：

    ```yaml
    test:
      needs: build
      runs-on: ubuntu-latest
    ```

1.  在生成作业部分，更新生成作业，在其中加入 **download-artifacts** 操作，使其与以下内容匹配：

    ```yaml
    steps:
    - uses: actions/checkout@v2
    - uses: actions/download-artifact@master
      with: 
        name: webpack artifacts
        path: public
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}
    - name: npm install, and test
      run: |
        npm install
        npm test
      env:
        CI: true
    ```

1.  在 **github-actions-for-ci** 存储库的 **“代码”** 选项卡（显示正在编辑的 **github-actions-for-ci/.github/workflows/node.js.yml** 文件的内容）上，单击右上角的 **“开始提交”**。
1.  在 **“提交更改”** 窗格上，接受默认设置（使用此设置时，更改将直接提交到当前分支），然后单击 **“提交更改”**。

> **备注**：提交之后，工作流将再次运行。 

> **备注**：我们的自定义工作流现在提供以下功能：

- 针对多个目标进行测试，以便我们知道受支持的操作系统和 Node.js 版本是否在正常运行
- 专用测试作业，以便我们将生成与测试详细信息区分开
- 对生成项目的访问权限，以便我们能够将这些项目部署到目标环境

#### 任务 13：与团队共享改进的 CI 工作流

在此任务中，你会将拉取请求和改进的工作流合并到主分支。

1.  导航回 **github-actions-for-ci GitHub** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面，在 **“对话”** 选项卡上，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面上，在 **github-learning-lab** 机器人的最新注释（标记为 **“合并 CI”**）中，查看 **“步骤 13: 与团队共享改进的 CI 工作流”** 部分。

    > **备注**：以下是我们到目前为止已实现的需求列表的状态：

    - 针对多个目标进行测试，以便我们知道受支持的操作系统和 Node.js 版本是否在正常运行
    - 专用测试作业，以便我们将生成与测试详细信息区分开
    - 对生成项目的访问权限，以便我们能够将这些项目部署到目标环境

    > **备注**：在接下来的几个步骤中，你将进行更改，以便为团队的工作流提供以下功能：

    - 分支保护，以确保无法删除或意外破坏主分支
    - 必要的审查，以便团队成员可以复核任意拉取请求
    - 显式批准，这样我们就可以快速合并，并可能自动完成合并和部署

1.  向下滚动到 **“批准的更改”** 部分，然后依次单击 **“合并拉取请求”** 和 **“确认合并”**。

#### 任务 14：自动执行审查流程

在此任务中，你将添加新的工作流文件来自动执行团队的审查流程

1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“改进 CI #5”** 页面上，在 **github-learning-lab** 机器人的最新注释中，单击 **“下一步”** 链接。这会将你重定向到名为 **“自定义工作流 #6”** 的拉取请求的 **“对话”** 选项卡。
1.  在 **“自定义工作流 #6”** 拉取请求的 **“对话”** 选项卡上，向下滚动并查看标有 **“步骤 14: 自动执行审查流程”** 的部分。

    > **备注**：GitHub Actions 可以针对不同的事件触发器运行多个工作流。我们将创建一个新的批准工作流，用于与我们的 Node.js 工作流配合使用。

1.  在标记为 **“活动: 添加新工作流文件以自动执行团队的审查流程”** 的部分的操作列表中,单击步骤 1 中的 **“新建文件”** 链接。这会将你重定向到 **github-actions-for-ci** 存储库的 **“代码”** 选项卡，其中显示了编辑器页面和打开的 **github-actions-for-ci/.github/workflows/approval-workflow.yml** 文件。
1.  在编辑器窗格中，输入 `name: AZ-400 Team approval workflow`，向下滚动到页面底部，然后单击 **“提交新文件”**

#### 任务 15：使用自动执行拉取请求审查的操作

在此任务中，你将在新工作流中使用社区操作

> **备注**：可以将工作流配置为：

- 使用事件从 GitHub 中运行
- 在计划的时间运行
- 在 GitHub 外部的事件发生时运行

> **备注**：到目前为止，我们对 Node.js 工作流使用了推送事件。当要对存储库的代码变更采取操作时，这很有意义。对于审查工作流，我们希望融入人工审查。例如，我们将使用“标签批准的拉取请求”操作，以便可以轻松查看在何时完成了合并拉取请求所需的审查。让我们使用 **pull_request_review** 事件来触发该操作，以准备好审查工作流。

1.  导航回拉取请求 **“自定义工作流 #6”** 的 **“对话”** 选项卡，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡中标记为 **“自定义工作流 #6”** 的页面上，在 **github-learning-lab** 机器人的最新注释中查看 **“步骤 15: 使用自动执行拉取请求审查的操作”** 部分。
1.  请注意，在本步骤中，只需将 **on: pull_request_review** 添加到在上一步中创建的工作流文件。
1.  在 **“步骤 15: 使用自动执行拉取请求审查的操作”** 部分,单击 **“提交建议”**，然后在弹出窗格中单击 **“提交更改”**。

#### 任务 16：在新工作流中创建审批作业

在此任务中，你将在新的工作流文件中创建一个新作业，该作业将使用社区操作

1.  在拉取请求 **“自定义工作流 #6”** 的 **“对话”** 选项卡上，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“自定义工作流 #6”** 页面上，在 **github-learning-lab** 机器人的最新注释中查看 **“步骤 16: 在新工作流中创建审批作业”** 部分。
1.  请注意，本步骤涉及按照以下格式，向在上一个任务中修改的同一工作流文件添加名为 **labelWhenApproved** 的新作业：

    ```yaml
    jobs:
      labelWhenApproved:
        runs-on: ubuntu-latest
    ```

1.  在 **“步骤 16: 在新工作流中创建审批作业”** 部分中，单击 **“提交建议”**，然后在弹出窗格中单击 **“提交更改”**。

#### 任务 17：自动审批

在此任务中，你将使用社区操作来自动执行部分审批流程

> **备注**：在上一个任务中，你创建了带有 **labelWhenApproved** 标识符的作业。现在，你将使用社区创建的名为 **pullreminders/label-when-approved-action** 的操作。达到预设的批准数后，该操作会向所有拉取请求添加一个标签。可将这些标签用作可视指示符，指示团队已准备好进行合并，甚至可以通过此方式来使用其他操作或工具，让其在收到所需数量的批准时自动合并拉取请求。

1.  在拉取请求 **“自定义工作流 #6”** 的 **“对话”** 选项卡上，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡的 **“自定义工作流 #6”** 页面上，在 **github-learning-lab** 机器人的最新注释中查看 **“步骤 17: 自动审批”** 部分。

    > **备注**：要实现自动审批，请遵循以下指南：

    - 工作流文件需要 **“steps:”** 块
    - 步骤名称是任意名称
    - 要使用社区操作，请加入 **“uses:”** 关键字
    - **label-when-approved-action** 需要使用名为 **“env:”** 的块以及以下环境变量：
       - **APPROVALS** 是应用标签所需的批准数。对于本实验室，请将其设置为 **1**。
       - **GITHUB_TOKEN** 是必需的，以便操作可以创建标签并将其应用于此存储库。 
       - **ADD_LABEL** 是达到批准数时应添加的标签的名称。其名称是任意名称。

1.  切换到 **github-actions-for-ci** 存储库的 **“代码”** 选项卡，在分支列表中，选择 **“team-workflow”** 条目，在文件夹列表中，单击 **“github/workflows”**，在文件列表中，单击 **“approval-workflow.yml”**，并在查看  **github-actions-for-ci/.github/workflows/approval-workflow.yml** 的内容时，单击代码窗格右侧的铅笔图标，以进入编辑模式。
1.  将以下内容添加到 **approval-workflow.yml** 文件。

    ```yaml
    steps:
    - name: Label when approved
      uses: pullreminders/label-when-approved-action@master
      env:
        APPROVALS: "1"
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        ADD_LABEL: "approved"
    ```

1.  单击 **“开始提交”**。
1.  在 **“提交更改”** 窗格上，接受默认设置（使用此设置时，更改将直接提交到当前分支），然后单击 **“提交更改”**。

#### 任务 18：使用分支保护

在此任务中，你将通过保护主分支来完成自动审查流程。

> **备注**：受保护的分支可确保存储库中的协作者无法对分支进行不可撤销的更改。通过启用受保护的分支，还可启用其他可选检查和要求，例如必需的状态检查和必需的审查。

1.  导航回拉取请求 **“自定义工作流 #6”** 的 **“对话”** 选项卡，向下滚动到 **github-learning-lab** 机器人的最新注释并查看其内容。
1.  在 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡中标记为 **“自定义工作流 #6”** 的页面上，在 **github-learning-lab** 机器人的最新注释中查看 **“步骤 18: 使用分支保护”** 部分。
1.  向上回滚到页面顶部，单击顶部菜单中的 **“设置”** 标头，然后单击左侧垂直菜单中的 **“分支”**。
1.  在 **“分支保护规则”** 部分，单击 **“添加规则”**。
1.  在 **“分支保护规则”** 的 **“分支名称模式”** 中键入 **“主”**，在 **“保护匹配的分支”** 设置列表中，启用 **“合并前需要拉取请求审查”**、**“合并前需要通过状态检查”**，选中每个生成和测试作业旁边的复选框，然后滚动到页面底部并单击 **“创建”**。
1.  导航回拉取请求 **“自定义工作流 #6”** 的 **“对话”** 选项卡，向下滚动到 **github-learning-lab** 机器人的最新注释，并查看 **“步骤 18: 使用分支保护”** 部分，然后在步骤 8 的 **“活动: 通过保护主分支来完成自动审查流程”小节中，单击“批准请求的审查”** 链接。这会将浏览器会话重定向到 **“自定义工作流”** 拉取请求的 **“已更改的文件”** 选项卡。
1.  在 **“自定义工作流”** 拉取请求的 **“已更改的文件”** 选项卡上，单击 **“查看更改”**，选择 **“批准”** 选项，然后单击 **“提交审查”**。这会将浏览器会话重定向回 **github-actions-for-ci** 存储库的 **“拉取请求”** 选项卡面上标记为 **“自定义工作流 #6”** 的页面，可通过 **github-learning-lab** 机器人的最新注释确认工作流是否已成功完成。

## 回顾

在本实验室中，你了解了如何设置 GitHub 工作流来自动执行生成流程。