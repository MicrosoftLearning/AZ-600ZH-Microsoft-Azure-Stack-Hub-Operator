---
lab:
    title: '实验室：在 Azure Stack Hub 中实现 SQL Server 资源提供程序'
    module: '模块 2：提供服务'
---

# 实验室 - 在 Azure Stack Hub 中实现 SQL Server 资源提供程序
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

150 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要允许租户部署 SQL Server 数据库。 

## 目标

完成本实验室后，你将能够：

- 在 Azure Stack Hub 中实现 SQL Server 资源提供程序。

## 实验室环境 

本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

实验室环境具有以下配置：

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

在本实验室课程中，你将安装通过 PowerShell 管理 Azure Stack Hub 所需的软件。 


### 练习 1：在 Azure Stack Hub 中安装 SQL Server 资源提供程序

在本练习中，你将在 Azure Stack Hub 中安装 SQL Server 资源提供程序。

1. 下载 SQL Server 资源提供程序二进制文件 
1. 安装 SQL Server 资源提供程序
1. 验证 SQL Server 资源提供程序的安装

>**备注**： 为了将此练习的持续时间缩至最短，已完成安装 Azure Stack Hub SQL Server 资源提供程序所需的一些任务，包括：

- 实现 Azure 市场联合
- 从 Azure 市场下载 **Microsoft AzureStack 附加产品 RP Windows Server**

#### 任务 1：下载 SQL Server 资源提供程序二进制文件

在此任务中，你将：

- 下载 SQL Server 资源提供程序二进制文件

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在与 **AzSHOST-1** 的远程桌面会话中，在显示 Azure Stack 管理员门户的 Web 浏览器的“中心”菜单中，单击 **“所有服务”**。
1. 在 **“所有服务”** 边栏选项卡上，搜索并选择 **“市场管理”**
1. 在“市场管理”边栏选项卡上，验证可用服务列表中是否显示了 **Microsoft AzureStack 附加产品 RP Windows Server**。
1. 在与 **AzSHOST-1** 的远程桌面会话中，启动另一个 Web 浏览器窗口，从 (https://aka.ms/azshsqlrp11931) 下载 SQL 资源提供程序自解压缩可执行文件，然后将其内容解压缩到 **C:\\Downloads\\SQLRP** 文件夹（需先创建该文件夹）。


#### 任务 2：安装 SQL Server 资源提供程序

在此任务中，你将：

- 安装 SQL Server 资源提供程序

1. 在与 **AzSHOST-1** 的远程桌面会话中，以管理员身份启动 Windows PowerShell。

    > **备注：** 确保启动新的 PowerShell 会话。

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，将 PowerShell 库配置为受信任的存储库

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，安装 SQL Server 资源提供程序所需的 AzureRM.Bootstrapper 模块版本：

    ```powershell
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose

    Install-Module -Name AzureRm.BootStrapper -RequiredVersion 0.5.0 -Force
    Install-Module -Name AzureStack -RequiredVersion 1.6.0
    ```

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，注册 Azure Stack Hub 操作员 PowerShell 环境：

    ```powershell
    Add-AzureRmEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. 从 **“Administrator: Windows PowerShell”** 窗口运行以下命令，设置当前环境：

    ```powershell
    Set-AzureRmEnvironment -Name 'AzureStackAdmin'
    ```

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，对当前环境进行身份验证（出现提示时，以 **CloudAdmin@azurestack.local** 用户身份使用密码 **Pa55w.rd1234** 进行登录）：

    ```powershell
    Connect-AzureRmAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，验证是否通过了身份验证，以及是否设置了相应的上下文：

    ```powershell
    Get-AzureRmContext
    ```

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，设置安装 SQL Server 资源提供程序所需的变量：

    ```powershell
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $downloadDir = 'C:\Downloads\SQLRP'

    # 设置 AzureStack\AzureStackAdmin 凭据
    $serviceAdmin = 'AzureStackAdmin@azurestack.local'
    $serviceAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $serviceAdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $serviceAdminPass)

    # 设置 AzureStack\CloudAdmin 凭据
    $cloudAdminName = 'AzureStack\CloudAdmin'
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object PSCredential($cloudAdminName, $cloudAdminPass)

    # 设置新资源提供程序 VM 本地管理员帐户的凭据
    $vmLocalAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ('sqlrpadmin', $vmLocalAdminPass)

    # 设置一个密码，该密码将保护生成的自签名证书的私钥，从而保护 SQL Server 资源提供程序
    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    # 更新 PowerShell 模块路径环境变量，以添加 SQL Server 资源提供程序模块
    $rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
    $env:PSModulePath = $env:PSModulePath + ';' + $rpModulePath 
    ```

1. 从 **“Administrator: Windows PowerShell”** 提示符，将当前目录更改为解压缩后的 SQL Server 资源提供程序安装文件所在的位置，然后运行 DeploySQLProvider.ps1 脚本：

    ```powershell
    Set-Location -Path 'C:\Downloads\SQLRP'

    .\DeploySQLProvider.ps1 `
        -AzCredential $serviceAdminCreds `
        -VMLocalCredential $vmLocalAdminCreds `
        -CloudAdminCredential $cloudAdminCreds `
        -PrivilegedEndpoint $privilegedEndpoint `
        -DefaultSSLCertificatePassword $pfxPass
    ```

    > **备注：** 等待安装完成。该操作需要约 1 小时。

#### 任务 3：验证 SQL Server 资源提供程序的安装

在此任务中，你将：

- 验证 SQL Server 资源提供程序的安装

1. 在与 **AzS-HOST1** 的远程桌面会话中，切换到显示 Azure Stack 管理员门户的 Web 浏览器窗口，单击“中心”菜单中的 **“资源组”**。 
1. 在 **“资源组”** 边栏选项卡上，单击 **system.local.sqladapter**。
1. 在 **system.local.sqladapter** 边栏选项卡上，查看 **“部署”** 条目并验证所有部署是否成功。
1. 在 Azure Stack 管理员门户中，导航到 **“虚拟机”** 并验证 SQL 资源提供程序 VM 是否已成功创建并正在运行。
1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 租户门户](https://portal.local.azurestack.external/)的 Web 浏览器窗口，如果出现提示，以 CloudAdmin@azurestack.local 身份登录。
1. 在 Azure Stack Hub 租户门户的中心菜单中，单击 **“创建资源”**。
1. 在 **“新建”** 边栏选项卡上，选择 **“数据 + 存储”**，然后验证可用资源类型列表中是否显示了 **SQL 数据库**。

>**回顾**： 在本练习中，你在 Azure Stack Hub 中安装了 SQL Server 资源提供程序


### 练习 2：在 Azure Stack Hub 中配置 SQL Server 资源提供程序

在本练习中，你将在 Azure Stack Hub 中配置 SQL Server 资源提供程序。

1. 为 SQL Server 托管服务器创建计划、套餐和订阅（以云操作员的身份）
1. 部署将成为 SQL Server 托管服务器的 Azure Stack Hub VM（以云操作员的身份）
1. 添加 SQL 托管服务器（以云操作员的身份）
1. 向用户提供 SQL 数据库（以云操作员的身份）
1. 创建 SQL 数据库（以用户身份）

>**备注**： 为了将此练习的持续时间缩至最短，已完成有助于配置 Azure Stack Hub SQL Server 资源提供程序的一些任务，包括：

- 从 Azure 市场下载 SQL Server 映像
- 从 Azure 市场下载 **Sql IaaS VM 扩展**


#### 任务 1：为 SQL Server 托管服务器创建计划、套餐和订阅（以云操作员的身份）

在此任务中，你将：

- 为 SQL Server 托管服务器创建计划、套餐和订阅（以云操作员的身份）

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“+ 创建资源”**。
1. 在 **“新建”** 边栏选项卡上，单击 **“套餐 + 计划”**，然后单击 **“计划”**。
1. 在 **“新建计划”** 边栏选项卡的 **“基本信息”** 选项卡上，指定以下设置：

    - 显示名称： **sql-server-hosting-plan1**
    - 资源名称： **sql-server-hosting-plan1**
    - 资源组：新资源组名称 **sql-server-hosting-plans-RG**

1. 单击 **“下一步:  服务 >”**。
1. 在 **“新建计划”** 边栏选项卡的 **“服务”** 选项卡上，选中 **“Microsoft.Compute”**、 **“Microsoft.Storage”** 和 **“Microsoft.Network”** 复选框。
1. 单击 **“下一步:  配额>”**。
1. 在 **“新建计划”** 边栏选项卡的 **“配额”** 选项卡上，指定以下设置：

    - Microsoft.Compute： **默认配额**
    - Microsoft.Network： **默认配额**
    - Microsoft.Storage： **默认配额**

1. 单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，返回到 **“新建”** 边栏选项卡，单击 **“套餐”**。
1. 在 **“新建套餐”** 边栏选项卡的 **“基本信息”** 选项卡中，指定以下设置：

    - 显示名称： **sql-server-hosting-offer1**
    - 资源名称： **sql-server-hosting-offer1**
    - 资源组： **sql-server-hosting-offers-RG**
    - 公开提供此套餐： **否**

1. 单击 **“下一步: 基本计划 >”**。 
1. 在 **“新建套餐”** 边栏选项卡的 **“基本计划”** 选项卡上，选中 **“sql-server-hosting-plan1”** 条目旁边的复选框。
1. 单击 **“下一步: 附加产品计划 >”**。
1. 保留 **“附加产品计划”** 设置为默认值，单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，返回到 **“新建”** 边栏选项卡，单击 **“订阅”**。
1. 在 **“创建用户订阅”** 边栏选项卡上，指定以下设置，然后单击 **“创建”**。

    - 名称： **sql-server-hosting-subscription1**
    - 用户： **cloudadmin@azurestack.local**
    - 目录租户： **ADFS.azurestack.local**
    - 套餐名称： **sql-server-hosting-offer1**

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。

1. 使 Azure Stack Hub 管理员门户窗口处于打开状态。


#### 任务 2：部署将成为 SQL Server 托管服务器的 Azure Stack Hub VM（以云操作员的身份）

在此任务中，你将：

- 部署将成为 SQL Server 托管服务器的 Azure Stack Hub VM（以云操作员的身份）

    >**备注**： 应在可计费用户订阅中创建作为 SQL Server 托管服务器运行的 Azure Stack Hub VM。

1. 在与 **AzS-HOST1** 的远程桌面会话中，切换到显示 [Azure Stack Hub 租户门户](https://portal.local.azurestack.external/)的 Web 浏览器窗口。
1. 在 Azure Stack Hub 租户门户的中心菜单中，单击 **“创建资源”**。
1. 在 **“新建”** 边栏选项卡上，选择 **“计算”**，然后在可用资源类型列表中，选择 **“{WS-BYOL} 免费 SQL Server 许可证: Windows Server 2016 上的 SQL Server 2017 Express”**。
1. 在 **“创建虚拟机”** 边栏选项卡的 **“基本信息”** 窗格上，指定以下设置并单击 **“确定”** （其他设置保留默认值）：

    - 名称： **sql-host-vm0**
    - VM 磁盘类型： **高级 SSD**
    - 用户名： **sqladmin**
    - 密码： **Pa55w.rd**
    - 订阅： **sql-server-hosting-subscription1**
    - 资源组： **新资源组名称 sql-server-hosting-RG**
    - 位置： **本地**

1. 在 **“选择大小”** 边栏选项卡上选择 **“DS1_v2”**，然后单击 **“选择”**。
1. 在 **“创建虚拟机”** 边栏选项卡的 **“设置”** 窗格上，将 **“网络安全组”** 设置设为 **“高级”**，然后单击 **“网络安全组(防火墙)”**。
1. 在 **“创建网络安全组”** 边栏选项卡中，单击 **“+ 添加入站规则”**。
1. 在 **“添加入站安全规则”** 边栏选项卡中，指定以下设置并单击 **“添加”** （将其他设置保留为默认值）：

    - 目标端口范围： **1433**
    - 协议： **TCP**
    - 操作： **允许**
    - Name: **SQL**

1. 回到 **“创建网络安全组”** 边栏选项卡，单击 **“确定”**。
1. 回到 **“创建虚拟机”** 边栏选项卡的 **“设置”** 窗格，指定以下设置并单击 **“确定”** （其他设置保留默认值）：

    - 启动诊断：禁用
    - 来宾 OS 诊断：禁用

1. 在 **“创建虚拟机”** 边栏选项卡的 **“SQL Server 设置”** 窗格上，指定以下设置并单击 **“确定”** （其他设置保留默认值）：

    - SQL 连接： **公共 (Internet)**
    - 端口： **1433**
    - SQL 身份验证： **启用**
    - 登录名： **SQLAdmin**
    - 密码： **Pa55w.rd**
    - 存储配置： **常规**
    - 自动修补： **禁用**
    - 自动备份： **禁用**
    - Azure 密钥保管库集成： **禁用**

1. 在 **“创建虚拟机”** 边栏选项卡的 **“摘要”** 窗格中，单击 **“确定”**。

    >**备注**： 等待部署完成。该操作需要约 20 分钟。

1. 部署完成后，导航到 **“sql-host-vm0”** 虚拟机边栏选项卡，在 **“概述”** 部分中的 **“DNS 名称”** 标签下，单击 **“配置”**。
1. 在 **“sql-host-vm0-ip \| 配置”** 边栏选项卡上的 **“DNS 名称标签(可选)”** 文本框中，键入 **sql-host-vm0** 并单击 **“保存”**。

    >**备注**： 这样即可通过 **sql-host-vm0.local.cloudapp.azurestack.external** DNS 名称使用 **sql-host-vm0**。

1. 在 **“sql-host-vm0-ip \| “配置”** 边栏选项卡上，将 **“分配”** 选项设置为 **“静态”**，然后单击 **“保存”**。

    >**备注**： 这将触发 **sql-host-vm0** 虚拟机的重启。


#### 任务 3：添加 SQL 托管服务器（以云操作员的身份）

在此任务中，你将：

- 添加 SQL 托管服务器（以云操作员的身份）

1. 在与 **AzSHOST-1** 的远程桌面会话中，在显示 Azure Stack 管理员门户的 Web 浏览器中，单击 **“所有服务”**，在 **“管理资源”** 部分，单击 **“SQL 托管服务器”**。

    > **备注：** 你可能需要刷新显示 Azure Stack 管理员门户的浏览器页面，以显示 **SQL 托管服务器**资源类型。

1. 在 **“SQL 托管服务器”** 边栏选项卡中，单击 **“+ 添加”**。
1. 在 **“添加 SQL 托管服务器”** 边栏选项卡上，指定以下设置：

    - SQL Server 名称： **sql-host-vm0.local.cloudapp.azurestack.external**
    - 用户名： **sqladmin**
    - 密码： **Pa55w.rd**
    - 托管服务器大小 (GB)： **50**
    - Always On 可用性组：未勾选
    - 订阅： **默认提供商订阅**
    - 资源组：新资源组名称 **sql.resources-RG**
    - 位置： **本地**

1. 在 **“添加 SQL 托管服务器”** 边栏选项卡上，单击 **“SKU”**，在 **“SKU”** 边栏选项卡上，单击 **“新建 SKU”**，然后在 **“创建 SKU”** 边栏选项卡上，指定以下设置：

    - Name: **MSSQL2017Exp**
    - 系列：**SQL Server 2017**
    - 层：**独立**
    - 版本：**Express**

1. 在 **“创建 SKU”** 边栏选项卡上，单击 **“确定”**，然后返回到 **“添加 SQL 托管服务器”** 边栏选项卡，单击 **“创建”**。

    > **备注：** 等待操作完成。所需时间应该不超过一分钟。

1. 在 **“SQL 托管服务器”** 边栏选项卡上，单击 **“刷新”**，并验证服务器列表中是否显示了 **sqlhost1.local.cloudapp.azurestack.external**。


#### 任务 4：向用户提供 SQL 数据库（以云操作员的身份）

在此任务中，你将：

- 向用户提供 SQL 数据库（以云操作员的身份）

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“+ 创建资源”**。
1. 在 **“新建”** 边栏选项卡上，单击 **“套餐 + 计划”**，然后单击 **“计划”**。
1. 在 **“新建计划”** 边栏选项卡的 **“基本信息”** 选项卡上，指定以下设置：

    - 显示名称： **sql-server-2017-express-db-plan1**
    - 资源名称： **sql-server-2017-express-db-plan1**
    - 资源组：新资源组名称 **sqldb-plans-RG**

1. 单击 **“下一步: 服务 >”**。
1. 在 **“新建计划”** 边栏选项卡的 **“服务”** 选项卡上，选中 **“Microsoft.SQLAdapter”** 复选框。
1. 单击 **“下一步: 配额>”**。
1. 在 **“新建计划”** 边栏选项卡的 **“配额”** 选项卡上，单击 **“Microsoft.SQLAdapter”** 条目旁边的 **“新建”**。
1. 在 **“创建配额”** 边栏选项卡上，指定以下设置，然后单击 **“创建”**：

    - 配额名称： **sql-server-2017-express-db-quota1**
    - 所有数据库最大大小 (GB)：**2**
    - 最大数据库数：**20**

1. 单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，返回到 **“新建”** 边栏选项卡，单击 **“套餐”**。
1. 在 **“新建套餐”** 边栏选项卡的 **“基本信息”** 选项卡中，指定以下设置：

    - 显示名称： **sql-server-2017-express-db-offer1**
    - 资源名称： **sql-server-2017-express-db-offer1**
    - 资源组： **sqldb-offers-RG**
    - 公开提供此套餐： **是**

1. 单击 **“下一步: 基本计划 >”**。 
1. 在 **“新建套餐”** 边栏选项卡的 **“基本计划”** 选项卡上，选中 **“sql-server-2017-express-db-plan1”** 条目旁边的复选框。
1. 单击 **“下一步: 附加产品计划 >”**。
1. 保留 **“附加产品计划”** 设置为默认值，单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。


#### 任务 5：创建 SQL 数据库（以用户身份）

在此任务中，你将：

- 创建测试用户帐户
- 使用新创建的用户帐户创建 SQL 数据库

1. 在与 **AzS-HOST1** 的远程桌面会话中，单击 **“开始”**，在 **“开始”** 菜单中，单击 **“Windows 管理工具”**，然后在管理工具列表中，双击 **“Active Directory 管理中心”**。
1. 在 **“Active Directory 管理中心”** 控制台中，单击 **“azurestack(本地)”**。
1. 在“详细信息”窗格中，双击 **“用户”** 容器。
1. 在 **“任务”** 窗格的 **“用户”** 部分中，单击 **“新建” -> “用户”**。
1. 在 **“创建用户”** 窗口中，指定以下设置，然后单击 **“确定”**： 

    - 全名：**T1U1**
    - 用户 UPN 登录名： **t1u1@azurestack.local**
    - 用户 SamAccountName： **azurestack\t1u1**
    - 密码： **Pa55w.rd**
    - 密码选项： **“其他密码选项”->“密码永不过期”**

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 Web 浏览器的 InPrivate 会话。
1. 在 Web 浏览器窗口中，导航到 [Azure Stack Hub 用户门户](https://portal.local.azurestack.external)，并使用 **t1u1@azurestack.local** 和密码 **Pa55w.rd** 登录。
1. 在 Azure Stack Hub 用户门户中，单击“仪表板”上的 **“获取订阅”** 磁贴。
1. 在 **“获取订阅”** 边栏选项卡的 **“名称”** 文本框中，键入 **“t1u1-sqldb-subscription1”**。
1. 在套餐列表中，选择 **“sql-server-2017-express-db-offer1”**，然后单击 **“创建”**。
1. 出现消息 **“订阅已创建。 必须刷新门户才能开始使用订阅”** 时，请单击 **“刷新”**。 
1. 在 Azure Stack Hub 租户门户的中心菜单中，单击 **“所有服务”**。
1. 在服务列表中，单击 **“SQL 数据库”**。
1. 在 **“SQL 数据库”** 边栏选项卡中，单击 **“+ 添加”**。
1. 在 **“创建数据库”** 边栏选项卡上，指定以下设置：

    - 数据库名称： **sqldb1**
    - 排序规则： **SQL_Latin1_General_CP1_CI_AS**
    - 最大大小 (MB)： **200**
    - 订阅： **t1u1-sqldb-subscription1**
    - 资源组：新资源组名称 **sqldb-RG**
    - 位置： **本地**
    - SKU： **MSSQL2017Exp**

    >**备注**： 你可能需要等待新创建的 SKU 在租户门户中可用。

1. 在 **“创建数据库”** 边栏选项卡中，单击 **“登录名”**。
1. 在 **“选择登录名”** 边栏选项卡中，单击 **“新建登录名”**。
1. 在 **“新建登录名”** 边栏选项卡上，指定以下设置，并单击 **“确定”**：

    - 数据库登录名： **dbAdmin**
    - 密码： **Pa55w.rd**

1. 返回 **“创建数据库”** 边栏选项卡，单击 **“创建”**。

    >**备注**： 等待部署完成。所需时间应该不超过一分钟。 

>**回顾**： 在本练习中，你将 SQL Server 托管服务器添加到了 Azure Stack Hub，将该服务器提供给了租户，并以租户用户身份部署了 SQL 数据库。