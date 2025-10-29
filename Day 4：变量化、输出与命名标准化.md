Day 4：变量化、输出与命名标准化
🎯 学习目标

学会用 variables / tfvars / locals 管理可变参数和命名规范。

掌握 output 的用法，将 Terraform 的部署结果可视化、可传递。

规范化标签（tags）与命名规则，为未来模块化和 CI/CD 做准备。

构建一个完整的、参数化的网络环境模板。

📘 一、为什么要变量化？

当你写的 Terraform 项目越来越大、环境越来越多（dev/sit/uat/prod），
就会出现以下问题：

❌ 手工改 .tf 文件参数 → 容易出错
❌ 命名规则不统一 → 运维混乱
❌ 无法快速切换环境 → 部署低效

👉 解决方案：变量 + 本地变量 + 输出

📗 二、文件结构（推荐）
terraform-azure-lab/
├── provider.tf
├── main.tf
├── variables.tf
├── locals.tf
├── outputs.tf
└── dev.tfvars

📙 三、定义变量 variables.tf
# variables.tf
variable "env" {
  type        = string
  description = "环境代号，例如 dev / sit / uat / prod"
}

variable "system" {
  type        = string
  description = "系统名，用于命名前缀"
}

variable "component" {
  type        = string
  description = "组件名，例如 network / app / db"
}

variable "location" {
  type        = string
  default     = "East US"
  description = "Azure 区域"
}

variable "owner" {
  type        = string
  description = "资源负责人邮箱"
}

variable "subnets" {
  description = "子网定义（名称→CIDR 映射）"
  type        = map(string)
  default = {
    "subnet-app" = "10.20.1.0/24"
    "subnet-db"  = "10.20.2.0/24"
  }
}

📘 四、定义本地变量（locals.tf）

locals 用来定义统一命名规则、标签（tags）、以及常用前缀组合。

# locals.tf
locals {
  # 统一命名前缀：环境 + 系统 + 组件
  name_prefix = lower(format("%s-%s-%s", var.env, var.system, var.component))

  # 公共标签（标签用于成本中心、审计、归属等）
  common_tags = {
    env        = var.env
    system     = var.system
    component  = var.component
    owner      = var.owner
    managed_by = "terraform"
  }
}


💡 好习惯：
每个组织都应该定义统一的命名和标签规则。
例如：

env：dev / sit / prod

system：系统名

component：模块名

owner：负责人邮箱

📘 五、在 main.tf 中应用变量与 locals
# main.tf

# 1️⃣ 创建资源组
resource "azurerm_resource_group" "rg" {
  name     = "${local.name_prefix}-rg"
  location = var.location
  tags     = local.common_tags
}

# 2️⃣ 创建虚拟网络
resource "azurerm_virtual_network" "vnet" {
  name                = "${local.name_prefix}-vnet"
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  address_space       = ["10.20.0.0/16"]
  tags                = local.common_tags
}

# 3️⃣ 创建子网（动态生成）
resource "azurerm_subnet" "subnets" {
  for_each             = var.subnets
  name                 = each.key
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [each.value]
}

📘 六、输出结果 outputs.tf
# outputs.tf

output "resource_group_name" {
  value       = azurerm_resource_group.rg.name
  description = "资源组名称"
}

output "vnet_name" {
  value       = azurerm_virtual_network.vnet.name
  description = "虚拟网络名称"
}

output "subnet_ids" {
  value       = { for k, s in azurerm_subnet.subnets : k => s.id }
  description = "子网 ID 映射"
}


执行后，Terraform 会在 CLI 自动输出这些结果。

📘 七、环境配置文件 dev.tfvars
# dev.tfvars
env       = "dev"
system    = "demo"
component = "network"
owner     = "zhen.liang@contoso.com"
location  = "East US"
subnets = {
  "subnet-app" = "10.20.1.0/24"
  "subnet-db"  = "10.20.2.0/24"
}

⚙️ 八、运行与验证
初始化环境
terraform init

预览执行计划（使用 dev.tfvars）
terraform plan -var-file=dev.tfvars


输出示例：

Plan: 3 to add, 0 to change, 0 to destroy.

应用变更
terraform apply -var-file=dev.tfvars -auto-approve


几分钟后：

你会在 Azure Portal 中看到：

一个资源组（dev-demo-network-rg）

一个虚拟网络（dev-demo-network-vnet）

两个子网（subnet-app 与 subnet-db）

查看输出结果
terraform output


输出示例：

resource_group_name = "dev-demo-network-rg"
vnet_name           = "dev-demo-network-vnet"
subnet_ids = {
  "subnet-app" = "/subscriptions/xxx/subnets/subnet-app"
  "subnet-db"  = "/subscriptions/xxx/subnets/subnet-db"
}

🧠 九、实践补充（命名规范与 for_each 扩展）

你可以通过修改 subnets 变量在 tfvars 中快速扩展网络：

subnets = {
  "frontend" = "10.20.1.0/24"
  "backend"  = "10.20.2.0/24"
  "db"       = "10.20.3.0/24"
}


Terraform 会智能识别差异，仅新增未存在的子网。
for_each 是生产环境中非常常见的写法（比 count 稳定得多）。

📊 十、变量化与命名规范的好处总结
分类	效果
可复用性	不同环境可通过 .tfvars 文件一键切换
可维护性	所有环境变量集中管理，命名一致
可审计性	统一标签让成本、日志分析更容易
可扩展性	支持 for_each 动态创建、locals 组合命名
可集成性	输出结果可被下游模块或 Pipeline 使用
✅ Day 4 成果验证清单
验证点	是否完成
通过 .tfvars 参数化环境配置	✅
locals 统一命名与标签规则	✅
for_each 成功批量创建多个子网	✅
output 正确显示关键资源信息	✅
理解变量、locals、output 关系	✅
🧩 思考练习

1️⃣ 试着在 locals.tf 中新增一个字段：

cost_center = "IT-Platform"


并将其加入 tags。

2️⃣ 复制 dev.tfvars → prod.tfvars，
只修改环境名与网段，再次执行部署。

3️⃣ 修改 subnets 中的网段，观察 Terraform Plan 是否提示资源替换（forces replacement）。