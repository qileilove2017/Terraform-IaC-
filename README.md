第1阶段：Terraform 基础与安装（第1周）
🎯 目标

理解 Terraform 核心概念与语法；掌握 CLI 命令与工作目录结构。

📚 学习内容

IaC（Infrastructure as Code）与 Terraform 基本理念

Terraform 与 ARM 模板区别

Terraform 工作原理（Init → Plan → Apply → Destroy）

Provider、Resource、Variable、Output、State

安装 Terraform 与 Azure CLI

🧩 实践任务

安装 Terraform 和 Azure CLI（macOS: Homebrew）

创建一个 provider.tf 文件连接 Azure：

provider "azurerm" {
  features {}
}


创建一个最小化示例：创建一个 resource group

resource "azurerm_resource_group" "rg" {
  name     = "learn-tf-rg"
  location = "East US"
}


执行 terraform init、terraform plan、terraform apply

✅ 验证成果

能解释 Terraform 工作流；

能在 Azure Portal 中看到新建的资源组。

第2阶段：核心语法与变量化设计（第2周）
🎯 目标

掌握 Terraform 配置语法、变量、输出与依赖关系。

📚 学习内容

Input Variables 与 Output Values

Locals 与 Data Source（引用已有资源）

Implicit & Explicit Dependencies

tfvars 文件与默认值

Terraform CLI 基本命令强化

🧩 实践任务

创建一个包含：

一个 Resource Group

一个 Storage Account（引用 RG）

一个 Virtual Network + Subnet

将 RG 名称、位置、前缀改用变量

使用 terraform.tfvars 管理参数

✅ 验证成果

运行 terraform apply -var-file=dev.tfvars；

输出资源信息。

第3阶段：模块化与状态管理（第3-4周）
🎯 目标

掌握模块化组织结构与远程状态存储（Azure Blob）。

📚 学习内容

Terraform 模块概念与目录结构

source 引用本地与远程模块

使用 backend "azurerm" 配置远程 state

State 加锁、版本控制策略

Output 与跨模块引用

🧩 实践任务

编写一个模块 /modules/network：

创建 vnet、subnet、nsg；

主文件引用该模块创建多个网络；

配置 state 存储在 Azure Blob：

terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

✅ 验证成果

能解释 state 管理与 drift；

能在 Storage Account 中查看 tfstate 文件。

第4阶段：企业级实践与安全集成（第5周）
🎯 目标

掌握 Terraform 与 Azure DevOps 集成、SPN 权限、安全配置。

📚 学习内容

Service Principal（SPN）身份认证

Key Vault 安全管理凭证

环境变量传递与 CI/CD 集成

Terraform Cloud 与 Azure DevOps Pipeline 集成

使用 terraform validate、fmt、tflint、pre-commit

🧩 实践任务

在 Azure DevOps 建立 Pipeline：

使用 terraform init/plan/apply 自动部署；

通过变量组注入 ARM_CLIENT_ID、ARM_CLIENT_SECRET；

创建 Key Vault 并通过 Terraform 获取机密；

使用 az ad sp create-for-rbac 创建 SPN；

用 pre-commit hook 检查格式与 lint。

✅ 验证成果

能全自动部署 + 审核；

无硬编码凭证；

Lint 检查通过。

第5阶段：进阶与架构最佳实践（第6周）
🎯 目标

掌握多环境、多订阅、模块复用、策略与组织级治理。

📚 学习内容

分层架构（Core Infra、Shared、App、Env 层）

Terraform Workspace 与多环境（dev/sit/uat/prod）

Reusable module pattern（Git 模块仓库）

Terraform import & drift detection

Terraform 结合 Azure Policy / Blueprints

🧩 实践任务

设计目录结构如下：

├── global/
│   └── modules/
│       ├── network/
│       ├── vm/
│       └── storage/
├── envs/
│   ├── dev/
│   ├── sit/
│   ├── uat/
│   └── prod/
└── pipelines/


让每个环境加载不同 tfvars；

使用 terraform workspace 切换；

编写 README 解释模块与环境间关系。

✅ 验证成果

实现多环境独立部署；

模块可复用；

能清晰解释团队协作策略。

📘 推荐资源

官方文档：https://developer.hashicorp.com/terraform

Microsoft Learn 路径：“Automate infrastructure deployments on Azure using Terraform”

GitHub 示例库：Azure/terraform

工具推荐：

VSCode 插件：Terraform、Azure Tools

Terraform-docs、tflint、tfsec

🎓 最终成果（学习完成后可独立实现）

✅ 企业级 Terraform 项目架构
✅ Pipeline 自动化部署 Azure Infra
✅ 模块化与多环境治理
✅ 安全凭证管理 + State 远程存储
✅ 能解释和优化 IaC 在团队协作中的设计