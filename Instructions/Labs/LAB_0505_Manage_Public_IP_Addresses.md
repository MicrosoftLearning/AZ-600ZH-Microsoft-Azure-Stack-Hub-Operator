---
lab:
    title: '实验室：在 Azure Stack Hub 中管理公共 IP 地址'
    module: '模块 5：管理基础结构'
---

# 实验室 - 在 Azure Stack Hub 中管理公共 IP 地址
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要管理公共 IP 地址资源。 

## 目标

完成本实验室后，你将能够：

- 管理公共 IP 地址资源

## 实验室环境

本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

实验室环境由以下部分组成：

- 在具有以下接入点的 **AzS-HOST1** 服务器上运行的 ASDK 部署：

  - 管理员门户：https://adminportal.local.azurestack.external
  - 管理员 ARM 终结点：https://adminmanagement.local.azurestack.external
  - 用户门户：https://portal.local.azurestack.external
  - 用户 ARM 终结点：https://management.local.azurestack.external

- 管理用户：

  - ASDK 云操作员用户名： **CloudAdmin@azurestack.local**
  - ASDK 云操作员密码： **Pa55w.rd1234**
  - ASDK 主机管理员用户名： **AzureStackAdmin@azurestack.local**
  - ASDK 主机管理员密码： **Pa55w.rd1234**

在本实验室课程中，你将安装通过 PowerShell 管理 Azure Stack Hub 所需的软件。你还将创建其他用户帐户。

## 说明

### 练习 0：实验室准备

在本练习中，你将创建将在本实验室中使用的 Active Directory 用户帐户：

1. 创建用户帐户（以云操作员的身份）

#### 任务 1：创建用户帐户（以云操作员的身份）

在此任务中，你将：

- 创建用户帐户（以云操作员的身份）

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，单击 **“开始”**，在 **“开始”** 菜单中，单击 **“Windows 管理工具”**，然后在管理工具列表中，双击 **“Active Directory 管理中心”**。
1. 在 **“Active Directory 管理中心”** 控制台中，单击 **“azurestack(本地)”**。
1. 在“详细信息”窗格中，双击 **“用户”** 容器。
1. 在 **“任务”** 窗格的 **“用户”** 部分中，单击 **“新建” -> “用户”**。
1. 在 **“创建用户”** 窗口中，指定以下设置，然后单击 **“确定”**： 

    - 全名： **T1U1**
    - 用户 UPN 登录名： **t1u1@azurestack.local**
    - 用户 SamAccountName： **azurestack\t1u1**
    - 密码：**Pa55w.rd**
    - 密码选项： **“其他密码选项” -> “密码永不过期”**

>**回顾**： 在本练习中，你创建了将在本实验室中使用的 Active Directory 帐户。


### 练习 1：创建套餐（以云操作员的身份）

在本练习中，你将充当云操作员。首先，你将查看公共 IP 地址的使用情况，然后创建一个包含网络服务的计划和一个包含该计划的套餐。接下来，你将公开此套餐，允许用户基于此套餐创建订阅。练习包含以下任务：

1. 查看公共 IP 地址的使用情况（以云操作员的身份）
1. 创建一个包含网络服务的计划（以云操作员的身份）。
1. 创建一个基于该计划的套餐并将其公开（以云操作员的身份）


#### 任务 1：查看公共 IP 地址的使用情况（以云操作员的身份）

在此任务中，你将：

- 查看公共 IP 地址的使用情况（以云操作员的身份）。

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在 Azure Stack Hub 管理员门户的 **“仪表板”** 页面上的 **“资源提供程序”** 磁贴中，单击 **“网络”**。 
1. 在 **“网络”** 边栏选项卡上，记下 **“公共 IP 池使用情况”** 图以及已使用和可用 IP 地址的数量。

#### 任务 2：创建一个包含网络服务的计划（以云操作员的身份）

在此任务中，你将：

- 创建一个包含网络服务的计划（以云操作员的身份）。

1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“+ 创建资源”**。 
1. 在 **“新建”** 边栏选项卡上，单击 **“套餐 + 计划”**。
1. 在 **“套餐 + 计划”** 边栏选项卡上，单击 **“计划”**。
1. 在 **“新建计划”** 边栏选项卡的 **“基本信息”** 选项卡上，指定以下设置：

    - 显示名称： **Network-plan1**
    - 资源名称： **network-plan1**
    - 资源组：新资源组名称 **network-plans-RG**

1. 单击 **“下一步:  服务 >”**。
1. 在 **“新建计划”** 边栏选项卡的 **“服务”** 选项卡上，选中 **“Microsoft.Network”** 复选框。
1. 单击 **“下一步: 配额>”**。
1. 在 **“新建计划”** 边栏选项卡的 **“配额”** 选项卡上，选择 **“新建”**。
1. 在 **“创建网络配额”** 边栏选项卡上，指定以下设置，然后单击 **“确定”**：

    - 名称： **Network-plan1-quota**
    - 虚拟网络数上限： **2**
    - 虚拟网络网关数上限： **2**
    - 网络连接数上限： **2**
    - 公共 IP 数上限： **20**
    - NIC 数上限： **20**
    - 负载均衡器数上限： **5**
    - 网络安全组数上限： **20**

1. 单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。


#### 任务 3：根据该计划创建一个套餐（以云操作员的身份）

在此任务中，你将：

- 根据该计划创建一个套餐（以云操作员的身份）

1. 在 Azure Stack Hub 管理员门户的主菜单中，单击 **“+ 创建资源”**。 
1. 在 **“新建”** 边栏选项卡上，单击 **“套餐 + 计划”**。
1. 在 **“套餐 + 计划”** 边栏选项卡上，单击 **“套餐”**。
1. 在 **“新建套餐”** 边栏选项卡的 **“基本信息”** 选项卡中，指定以下设置：

    - 显示名称： **Network-offer1**
    - 资源名称： **network-offer1**
    - 资源组：名为 **network-offers-RG** 的新资源组
    - 公开提供此套餐： **是**

1. 单击 **“下一步: 基本计划 >”**。 
1. 在 **“新建套餐”** 边栏选项卡的 **“基本计划”** 选项卡上，选中 **“Network-plan1”** 条目旁边的复选框。
1. 单击 **“下一步: 附加产品计划 >”**。
1. 保留 **“附加产品计划”** 设置为默认值，单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。

>**回顾**： 在本练习中，你创建了一个计划，并基于该计划创建了一个公共套餐。


### 练习 2：创建公共 IP 地址资源（以用户的身份）

在本练习中，你将充当注册了在第一个练习中创建的套餐，创建了新订阅并在该订阅中创建了公共 IP 地址资源的用户。练习包含以下任务：

1. 注册套餐（以用户的身份）
1. 连接到 Azure Stack Hub 用户 Azure 资源管理器终结点（以用户的身份）
1. 创建 IP 地址资源（以用户的身份）

#### 任务 1：注册套餐（以用户的身份）

在此任务中，你将：

- 注册套餐（以用户的身份）

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 Web 浏览器的 InPrivate 会话。
1. 在 Web 浏览器窗口中，导航到 [Azure Stack Hub 用户门户](https://portal.local.azurestack.external)，并使用 **t1u1@azurestack.local** 和密码 **Pa55w.rd** 登录。
1. 在 Azure Stack Hub 用户门户中，单击“仪表板”上的 **“获取订阅”**。
1. 在 **“获取订阅”** 边栏选项卡的 **“显示名称”** 文本框中，键入 **“T1U1-network-subscription1”**。
1. 在套餐列表中，选择 **“Network-offer1”**，然后单击 **“创建”**。
1. 在 **“订阅已创建。必须刷新门户才能开始使用订阅”** 消息框中，单击 **“刷新”**。


#### 任务 2：连接到 Azure Stack Hub Azure 资源管理器用户终结点（以用户的身份）

在此任务中，你将：

- 连接到 Azure Stack Hub Azure 资源管理器用户终结点（以用户的身份）

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell 7。

    >**备注**： 有关设置 PowerShell 与 Azure Stack Hub 的连接的详细说明，请按照实验室 **“通过 PowerShell 连接到 Azure Stack Hub”** 中的说明操作。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，安装本实验室所需的 Azure Stack Hub PowerShell 模块：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**备注**： 忽略有关已可用命令的任何错误消息。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，下载并解压缩 Azure Stack Hub 工具：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，注册 Azure Stack Hub 用户环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以 **t1u1@azurestack.local** 用户的身份通过浏览器会话启动对 Azure Stack Hub 用户环境的身份验证：

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser' -UseDeviceAuthentication
    ```

1. 在 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口中，查看生成的消息，在 InPrivate 模式下再打开一个 Web 浏览器窗口，导航到 [adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth) 页面，然后键入已查看的消息中包含的代码。如果出现提示，请以 **t1u1@azurestack.local** 用户的身份再次登录。
1. 切换回 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，并验证是否已成功以 **t1u1@azurestack.local** 用户的身份进行身份验证。


#### 任务 3：创建 IP 地址资源（以用户的身份）

在此任务中，你将：

- 创建 IP 地址资源（以用户的身份）

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，验证使用的是否是新预配的订阅： 

    ```powershell
    (Get-AzSubscription).Name
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，在当前订阅中注册网络资源提供程序：

    ```powershell 
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    ```

    >**备注**： 必须注册资源提供程序才能创建受该提供程序管理的资源。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，创建将托管公共 IP 地址资源的资源组：

    ```powershell 
    $rg = New-AzResourceGroup -Name publicIPs-RG -Location local
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，创建公共 IP 地址资源： 

    ```powershell
    1..5 | ForEach-Object {New-AzPublicIpAddress -Name "publicIP$_" -ResourceGroupName $rg.ResourceGroupName -AllocationMethod Static -Location local}
    ```

1. 等待所有 IP 地址资源预配完成。

>**回顾**：完成本练习后，你已在用户订阅中创建了公共 IP 地址资源。


### 练习 4：管理公共 IP 地址的使用情况（以云操作员的身份）

在本练习中，你将充当云操作员，查看和管理公共 IP 地址的使用情况。练习包含以下任务：

1. 查看公共 IP 地址的使用情况
2. 添加公共 IP 地址池

#### 任务 1：查看公共 IP 地址的使用情况（以云操作员的身份）

在此任务中，你将：

- 查看公共 IP 地址的使用情况（以云操作员的身份）。

1. 切换到显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口，以 CloudAdmin@azurestack.local 身份登录。
1. 在 Azure Stack Hub 管理员门户的主菜单中，单击 **“仪表板”**，然后在 **“资源提供程序”** 磁贴上，单击 **“网络”**。
1. 在 **“网络”** 边栏选项卡上，再次查看 **“公共 IP 池使用情况”** 图以及已使用和可用 IP 地址的数量。

    >**备注**：这些数字应该已更改，反映出你（以用户的身份）在用户订阅中创建的其他 5 个公共 IP 地址。

#### 任务 2：添加公共 IP 地址池（以云操作员的身份）

在此任务中，你将：

- 添加公共 IP 地址池

1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“网络”** 边栏选项卡上的 **“公共 IP 池使用情况”** 磁贴。

    >**备注**： 如果 **“公共 IP 池”** 边栏选项卡显示消息 **“排他操作‘启动’正在进行中。 该操作运行过程中，将禁用添加节点和添加 IP 池操作。 单击此处查看活动日志”**，则需要等待“启动”操作完成，然后再进行下一步。

1. 在 **“公共 IP 池”** 边栏选项卡中，单击 **“+ 添加 IP 池”**。 
1. 在 **“添加 IP 池”** 边栏选项卡上，指定以下设置并单击 **“添加”**。

    - 名称： **公共池 1**
    - 区域： **本地**
    - 地址范围（CIDR 块）： **192.168.110.0/24**

1. 等待更改生效，然后导航回 **“网络”** 边栏选项卡。
1. 查看 **“公共 IP 池使用情况”** 图，并记下可用 IP 地址的数量变化。

>**回顾**： 在本练习中，你查看并配置了公共 IP 地址池。
