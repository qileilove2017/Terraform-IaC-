Day 2：Terraform 工作机制与核心命令

今天我们正式上手 Terraform 的执行流程，
通过实践理解 Terraform 如何从代码生成真实的 Azure 资源。

🎯 学习目标

理解 Terraform 的工作目录结构与执行生命周期

掌握四个核心命令：init、plan、apply、destroy

了解执行计划（Plan）与状态文件（State）的关系

亲手部署并删除第一个 Azure 资源

📘 一、Terraform 工作机制

Terraform 是一个声明式的 IaC 工具，你告诉它“你想要的最终状态”，
Terraform 会自动计算“当前状态 → 目标状态”的差异，然后执行变更。

Terraform 工作原理流程如下：

1️⃣ 编写配置文件（.tf）
    ↓
2️⃣ terraform init
    初始化项目并下载 provider 插件
    ↓
3️⃣ terraform plan
    解析 .tf 文件，生成执行计划（Preview）
    ↓
4️⃣ terraform apply
    应用计划，实际在 Azure 创建资源
    ↓
5️⃣ terraform destroy
    销毁在 state 中记录的所有资源

📗 二、准备项目目录

在昨天的目录下继续操作：

cd ~/terraform-azure-lab


创建文件结构：

terraform-azure-lab/
├── provider.tf
└── main.tf

📙 三、定义第一个资源

在 main.tf 中添加以下内容：

# main.tf
resource "azurerm_resource_group" "rg" {
  name     = "tf-day2-rg"
  location = "East US"
}


解释：

resource：定义一个资源块。

"azurerm_resource_group"：资源类型（Azure Resource Group）。

"rg"：资源逻辑名称（在 state 中的引用名）。

name / location：Azure RG 的属性。

⚙️ 四、核心命令详解与实操
1️⃣ 初始化项目
terraform init


作用：

初始化工作目录；

下载 provider 插件；

生成 .terraform 目录；

创建初始 terraform.lock.hcl（依赖版本锁定文件）。

💡 如果你修改了 provider 版本，需要重新运行 terraform init -upgrade。

2️⃣ 预览执行计划
terraform plan


作用：

Terraform 会解析 .tf 文件；

比较当前状态（tfstate）与配置文件定义；

生成一个执行计划（不实际执行）。

示例输出：

Plan: 1 to add, 0 to change, 0 to destroy.


解释：

add：要新建的资源数量；

change：要更新的资源数量；

destroy：要删除的资源数量。

3️⃣ 执行计划（真正部署）
terraform apply


执行时 Terraform 会再次生成 plan 并要求确认：

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.


输入：

yes


几分钟后，你将在 Azure Portal 中看到名为 tf-day2-rg 的资源组。

4️⃣ 检查状态文件

Terraform 会自动生成：

terraform.tfstate


查看：

terraform show


或者查看资源列表：

terraform state list


输出示例：

azurerm_resource_group.rg


🧠 State 文件是 Terraform 的“记忆”，它记录了：

哪些资源已创建；

它们的属性值；

Terraform 与云环境的映射关系。

5️⃣ 销毁资源
terraform destroy


Terraform 会读取 state 文件，删除所有记录的资源。
输入 yes 确认后，Azure Portal 中的资源组将被删除。

💡 destroy 只删除 Terraform 管理的资源，Portal 手动创建的不受影响。

📈 五、执行命令总结表
命令	作用	是否修改资源	备注
terraform init	初始化目录、下载 provider	❌ 否	首次运行必须执行
terraform plan	预览将要执行的操作	❌ 否	可反复执行
terraform apply	实际创建或修改资源	✅ 是	自动更新 state
terraform destroy	删除所有 Terraform 管理的资源	✅ 是	慎用，需确认
terraform show	查看 state 文件内容	❌ 否	便于调试
🧩 六、Day 2 实践任务

1️⃣ 在本地执行：

terraform init
terraform plan
terraform apply


2️⃣ 打开 Azure Portal，验证资源组是否创建。
3️⃣ 执行：

terraform state list


查看 Terraform 管理的资源。
4️⃣ 最后执行：

terraform destroy


删除资源。

🧠 七、Day 2 思考题（掌握关键理解）

Terraform 如何判断要创建、修改或删除资源？

如果我手动在 Portal 中修改了资源，Terraform 的 plan 会显示什么？

为什么 Terraform 需要 state 文件，而不是每次都直接扫描 Azure？

答案（简略）：

Terraform 通过对比 state 文件与当前配置文件；

如果 Portal 改动，plan 会检测出 drift（偏移）；

因为扫描云资源耗时大、权限复杂、性能低。

✅ Day 2 成果验证
检查项	是否完成
Terraform init 成功执行	✅
Terraform plan 输出 “1 to add”	✅
Terraform apply 成功创建资源组	✅
Terraform destroy 成功删除资源	✅
理解 state 文件与 plan 的关系	✅