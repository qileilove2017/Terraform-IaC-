Day 1：认识 Terraform 与 IaC
🎯 学习目标

理解基础概念：什么是 IaC？Terraform 有什么作用？

掌握 Terraform 的基本组成结构与工作机制。

完成本地安装（Terraform + Azure CLI）。

通过 az login 登录 Azure 并验证环境。

📘 一、理解 IaC（Infrastructure as Code）

概念

Infrastructure as Code（基础设施即代码）是一种用代码定义和管理 IT 基础设施的理念。
传统上我们在 Azure Portal 中“点点点”创建资源，而 IaC 则通过代码描述想要的最终状态，Terraform 根据代码自动实现创建、修改、销毁资源。

示例对比

部署方式	操作特点	优缺点
手动操作（Portal）	人工创建资源	简单直观，但难以复现
脚本（CLI/Powershell）	命令式，按步骤执行	可自动化，但可维护性低
Terraform（IaC）	声明式，定义“最终状态”	可重现、可追踪、可协作

Terraform 的理念是：

你告诉它“我想要什么”，它负责“怎么实现”。

📗 二、Terraform 的核心组件
组件	作用
Provider	告诉 Terraform 管理哪个云平台（如 Azure、AWS、GCP）。
Resource	定义要创建的资源，如虚拟机、网络、数据库等。
Variable	变量，用于参数化配置。
Output	输出变量，用于显示或传递结果。
State	Terraform 的“大脑”，记录已创建资源状态。
📙 三、Terraform 工作流程（核心 4 步）
Init → Plan → Apply → Destroy

阶段	命令	说明
初始化	terraform init	初始化目录并下载 Provider 插件。
规划	terraform plan	模拟执行，显示将会创建/修改/删除的资源。
应用	terraform apply	执行部署操作，实际在 Azure 中创建资源。
销毁	terraform destroy	删除所有在 state 中记录的资源。

💡 Terraform 的执行是幂等的：
如果环境已经与代码一致，plan 会显示 “No changes required”。

🧩 四、安装与环境准备
1. 安装 Terraform（macOS）
brew tap hashicorp/tap
brew install hashicorp/tap/terraform


验证：

terraform -version

2. 安装 Azure CLI
brew install azure-cli


验证：

az version

3. 登录 Azure
az login


如果有多个订阅，设置当前订阅：

az account list -o table
az account set --subscription "<your-subscription-id>"

🧱 五、创建第一个 Terraform 项目目录

在你的工作区创建项目文件夹：

mkdir ~/terraform-azure-lab
cd ~/terraform-azure-lab


创建第一个文件 provider.tf

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}

🚀 六、初始化与验证

执行命令：

terraform init


输出结果示例：

Initializing provider plugins...
- Finding hashicorp/azurerm versions matching "~> 4.0"...
- Installing hashicorp/azurerm v4.2.0...
Terraform has been successfully initialized!


表示 Terraform 已成功安装并下载 Azure Provider。

🧠 七、理解 Terraform 的文件结构
文件	说明
provider.tf	定义使用的云平台及版本。
main.tf	定义要创建的资源（下一天会写）。
variables.tf	定义输入变量。
outputs.tf	定义输出结果。
terraform.tfstate	自动生成的状态文件。

建议：所有 .tf 文件可按功能拆分，Terraform 会自动合并。

🧪 八、Day 1 任务验证

✅ Terraform 与 Azure CLI 安装完成
✅ 成功登录 Azure (az login)
✅ 能执行 terraform init 并看到 provider 初始化成功
✅ 理解 IaC、Provider、Resource、State、Plan、Apply 等概念

🧩 九、扩展阅读（推荐）

📘 HashiCorp 官方入门文档：https://developer.hashicorp.com/terraform/intro

📗 Microsoft Learn 路径：“Automate infrastructure deployments on Azure using Terraform”

📘 视频资源（建议）：YouTube 搜索 “Terraform for Azure beginners”