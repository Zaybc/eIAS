# eIAS(experimental Internet Advertising System，试验性互联网广告系统)
Curriculum Design of Software Engineering Theory and Practice of Class 2, Grade 2021, Huazhong University of Science and Technology

软件工程理论与实践课程设计 华中科技大学 软件工程2102班

# Introduction项目介绍
*该项目的主要结构包括以下三个方面：*

*1.前端网站：该网站包含企业客户和服务企业管理员两个入口。企业客户可以在网站上注册、登录、注销，维护信息，预存广告费用，查询广告展示情况，购买广告展示服务，以及中止广告展示服务。服务企业管理员可以在网站上审核企业广告和客户信息，浏览和查询企业客户信息和广告，定价广告，以及管理应用。该网站需要具备良好的用户体验和易用性，能够满足企业客户和服务企业管理员的需求。*

*2.后端服务器：该服务器是网站的核心部分，负责处理用户请求，验证用户身份，处理交易，以及保存和检索广告数据等。该服务器需要具备高可用性，可扩展性和高安全性。*

*3.接口：该接口负责把交易网站与其他的web应用/移动app/微信小程序连接，以便把服务企业管理员审核通过的广告传输到预留的广告位置，开始提供广告服务。该接口需要具备良好的稳定性和高效性。*

*除了以上三个方面，该项目还需要涵盖其他一些重要的方面：*

- *数据库：用于保存企业客户和广告数据等关键信息。*
- *付款渠道：需要集成可靠的付款渠道，以便客户能够方便地支付广告费用。*
- *广告展示应用：用于展示广告，需要具备良好的用户体验和可靠性。*

*综上所述，该项目需要在前端网站，后端服务器，接口以及其他方面进行开发和部署，以便为企业客户和服务提供商提供高质量的广告服务。*

# Project manager项目负责人
*Zaybc      沈星志*
# Project member项目组成员
*lolowhg     袁开成
Alexhust1   敖轩
turqu2oise  姜毅冉*

# Features项目特性

*多角色交互：该项目涉及到多个角色之间的交互，包括企业客户、服务企业管理员和广告展示应用。他们分别拥有不同的权限和操作，需要进行权限控制和数据管理。*

*广告管理：该项目的核心是广告管理，包括审核、定价、购买和中止广告展示等。需要实现广告的展示和管理，以满足企业客户的需求。*

*交易管理：该项目需要处理企业客户的付款、退款和结算等交易操作，需要与第三方支付渠道进行集成。*

*实时数据查询：企业客户需要实时查询广告展示情况和效果，需要设计相关数据展示和查询功能。*

*安全性保障：该项目需要处理企业客户的敏感数据和付款信息，需要实现数据安全和保密措施，例如用户身份验证、数据加密和防止数据泄露等。*

*界面友好：该项目的用户群体广泛，需要设计简洁明了的界面，方便不同层级的用户使用。*


# CodingGuideline代码规范

**对于前端开发：**

1. *使用语义化的HTML标记*
2. *在CSS中使用合理的命名约定和组织方式*
3. *遵循单一职责原则，在JavaScript中尽量避免耦合性较强的代码*
4. *使用有意义的变量、函数和类名，并遵循驼峰命名法*
5. *在代码中添加注释以解释功能和实现方式*
6. *遵循代码重构的原则，尽可能保证代码的可维护性和可扩展性*
7. *遵循开放封闭原则，在添加新功能时尽量不修改原有代码*

**对于后端开发：**

1. *遵循RESTful API设计规范，使用合适的HTTP请求方法和状态码*
2. *使用有意义的变量、函数和类名，并遵循驼峰命名法*
3. *在代码中添加注释以解释功能和实现方式*
4. *遵循单一职责原则，在代码中尽量避免耦合性较强的代码*
5. *遵循代码重构的原则，尽可能保证代码的可维护性和可扩展性*
6. *遵循开放封闭原则，在添加新功能时尽量不修改原有代码*
7. *使用异常处理机制处理异常情况*
8. *避免在代码中硬编码敏感信息，如密码和密钥*
9. *对于数据库操作，使用参数化查询方式避免SQL注入攻击。*


# Version Control Guidelines版本管理规范
遵循以下步骤：

1. 每个开发者从主分支上创建自己的分支，例如 feature/xxx，各自在自己的分支上开发，以避免直接在主分支上进行修改而导致的代码冲突。
2. 开发者应该经常从主分支上拉取最新的代码，以便及时解决代码冲突。
3. 开发者应该尽可能频繁地提交代码，而不是等到一大堆代码修改后再一次性提交。这可以降低代码冲突的风险，并且能够更方便地进行代码回滚。
4. 每次需要更新代码时，先将主分支上的最新代码拉取到自己的分支，避免自己的分支过于落后。
5. 当自己的代码修改完成后，先将主分支上的最新代码合并到自己的分支，解决可能出现的冲突。然后再将自己的分支合并到主分支上，再由其他开发者进行代码审核和测试。
6. GitHub仓库已经设置了合并请求需求和审查(Require a pull request before merging)。每个开发者将代码首先fork到自己的库中，然后在自己的代码库中建立新分支，而后对自己的任务进行多分支小规模的处理。每解决一个完整需求后，通过合并请求（Pull Request）来将自己的代码合并到主分支上，这样可以方便地进行代码讨论和合并审核，同时尽量避免代码冲突。*
7. 如果不幸出现了代码冲突，开发者应该在本地解决冲突后再提交代码，以避免丢失代码和导致更严重的冲突。

遵循上述步骤可以有效地避免直接在同一个分支上进行修改而导致的冲突和代码丢失问题，保证代码的稳定性和可维护性。同时，也可以更好地掌控代码的版本和历史记录，方便进行回溯和修复。

*步骤6详细步骤：

1. Fork 你想参与的代码仓库。
2. 使用 `git clone` 命令将 fork 的仓库克隆到本地计算机。
3. 使用 `git checkout -b <branch-name>` 命令为更改创建一个新的分支。给分支起一个描述性的名称，反映你所做的更改。
4. 在本地仓库中更改代码。
5. 使用 `git commit -m 描述信息 -s 签名(就是你标识自己的名字)` 命令提交更改。请确保-m后面包括一条描述性的提交信息，说明你所做的更改
6. 使用 `git pull` 命令从源代码库中拉取最新的代码，并检查是否存在代码冲突。若存在冲突，则与其他组员交流并得出一致，确保自己的代码与库里的代码不存在冲突。
7. 使用 `git push` 命令将更改推送到你自己 GitHub 上 fork 的仓库中。
8. 转到 GitHub 上的主仓库页面，然后单击 "New pull request" 按钮。
9. 选择你 fork 的仓库和刚刚推送更改的分支。
10. 检查你所做的更改并确保它们无误。
11. 为你的拉取请求添加一个描述性的标题和说明。解释您所做的更改及其重要性。
12. 提交拉取请求，等待主仓库的维护人员审查并合并您的更改。

另：可以学习使用GUI的git管理工具，如：vscode中的源代码管理（搭配插件gitlens）或sourcetree软件。
