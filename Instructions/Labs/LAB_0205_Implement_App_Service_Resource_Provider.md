---
lab:
    title: '实验室：在 Azure Stack Hub 中实现应用服务资源提供程序'
    module: '模块 2：提供服务'
---

# 实验室 - 在 Azure Stack Hub 中实现应用服务资源提供程序
# 学生实验室手册

## 实验室依赖项

- 在 Azure Stack Hub 中实现 SQL Server 资源提供程序

## 预计用时

4 小时

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要允许租户部署应用服务应用和 Azure Functions。

## 目标

完成本实验室后，你将能够：

 - 在 Azure Stack Hub 中实现应用服务资源提供程序。

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


### 练习 1：在 Azure Stack Hub 中安装应用服务资源提供程序

在本练习中，你将在 Azure Stack Hub 中安装应用服务资源提供程序。

1. 预配 SQL Server 托管服务器
1. 预配文件服务器
1. 安装应用服务资源提供程序
1. 验证应用服务资源提供程序的安装

>**备注**： 为了最大限度地缩短此练习的时长，安装 Azure Stack Hub 应用服务资源提供程序所需的一些任务已完成，包括：

- 实现 Azure 市场联合
- 下载以下 Azure 市场项：

  - **[smalldisk] Windows Server 2019 数据中心服务器核心 - 自带许可**
  - **Windows Server 2016 数据中心 - 自带许可** 
  - **自定义脚本扩展**


#### 任务 1：预配 SQL Server 托管服务器

在此任务中，你将：

- 预配 SQL Server 托管服务器

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在 Azure Stack Hub 管理员门户的中心菜单中，单击 **“创建资源”**。
1. 在 **“新建”** 边栏选项卡上，选择 **“计算”**，然后在可用资源类型列表中，选择 **“{WS-BYOL} 免费 SQL Server 许可证: Windows Server 2016 上的 SQL Server 2017 Express”**。
1. 在 **“创建虚拟机”** 边栏选项卡的 **“基本信息”** 窗格上，指定以下设置并单击 **“确定”** （其他设置保留默认值）：

    - 名称：**SqlHOST1**
    - VM 磁盘类型：**高级 SSD**
    - 用户名： **sqladmin**
    - 密码：**Pa55w.rd**
    - 订阅：**默认提供商订阅**
    - 资源组：新资源组名称 **sql.resources-RG**
    - 位置： **本地**

1. 在 **“选择大小”** 边栏选项卡上选择 **“DS1_v2”**，然后单击 **“选择”**。
1. 在 **“创建虚拟机”** 边栏选项卡的 **“设置”** 窗格上，将 **“网络安全组”** 设置设为 **“高级”**，然后单击 **“网络安全组(防火墙)”**。
1. 在 **“创建网络安全组”** 边栏选项卡中，单击 **“+ 添加入站规则”**。
1. 在 **“添加入站安全规则”** 边栏选项卡中，指定以下设置并单击 **“添加”** （将其他设置保留为默认值）：

    - 目标端口范围： **1433**
    - 协议： **TCP**
    - 操作： **允许**
    - 优先级： **200**
    - 名称： **custom-allow-sql**

1. 回到 **“创建网络安全组”** 边栏选项卡，单击 **“确定”**。
1. 回到 **“创建虚拟机”** 边栏选项卡的 **“设置”** 窗格，指定以下设置并单击 **“确定”** （其他设置保留默认值）：

    - 启动诊断：禁用
    - 来宾 OS 诊断：禁用

1. 在 **“创建虚拟机”** 边栏选项卡的 **“SQL Server 设置”** 窗格上，指定以下设置并单击 **“确定”** （其他设置保留默认值）：

    - SQL 连接：**公共 (Internet)**
    - 端口：**1433**
    - SQL 身份验证：**启用**
    - 登录名： **SQLAdmin**
    - 密码：**Pa55w.rd**
    - 存储配置：**常规**
    - 自动修补：**禁用**
    - 自动备份：**禁用**
    - Azure 密钥保管库集成：**禁用**

1. 在 **“创建虚拟机”** 边栏选项卡的 **“摘要”** 窗格中，单击 **“确定”**。

    >**备注**： 等待部署完成。该操作需要约 20 分钟。

1. 部署完成后，导航到 **“SqlHOST1”** 虚拟机边栏选项卡，在 **“概述”** 部分的 **“DNS 名称”** 标签下，单击 **“配置”**。
1. 在 **“SqlHOST1-ip”\| “配置”** 边栏选项卡上的 **“DNS 名称标签(可选)”** 文本框中，键入 sqlhost1 并单击 **“保存”**。

    >**备注**： 这样可以通过 **sqlhost1.local.cloudapp.azurestack.external** DNS 名称提供 **sqlhost1**。

1. 在 **“sqlhost1-ip” \| “配置”** 边栏选项卡上，将 **“分配”** 选项设置为 **“静态”**，然后单击 **“保存”**。

    >**备注**： 这将触发 **sqlhost1** 虚拟机重启。请等到重启完成，再继续下一步。

1. 在与 **AzSHOST-1** 的远程桌面会话中，启动与 **sqlhost1.local.cloudapp.azurestack.external** 的远程桌面会话，并在出现提示时使用以下凭据登录：

    - 用户名： **SQLAdmin**
    - 密码： **Pa55w.rd**

1. 在与 **SqlHOST1** 的远程桌面会话中，右键单击 **“开始”**，然后在右键菜单中选择 **“命令提示符(管理员)”**。 
1. 在与 **SqlHOST1** 的远程桌面会话中，从 **“Administrator: Command Prompt”** 中运行以下命令以启动与本地 SQL Server 实例的 SQLCMD 会话：

    ```
    sqlcmd
    ```

1. 在与 **SqlHOST1** 的远程桌面会话中，从 **“Administrator: Command Prompt”** 中运行以下命令以启用 SQL Server 的已包含数据库身份验证：

    ```
    sp_configure 'contained database authentication', 1;
    GO
    RECONFIGURE;
    GO
    ```

    > **备注：** 在本实验室后面部分实现应用服务资源提供程序时，要使用此托管服务器，必须进行此操作。

    > **备注：** 将与 **sqlhost1.local.cloudapp.azurestack.external** 的远程桌面会话保持打开。你将在本实验室后面部分用到它。


#### 任务 2：预配文件服务器

在此任务中，你将：

- 预配文件服务器

1. 切换到与 **AzSHOST-1** 的远程桌面会话，在 Azure Stack 管理员门户的中心菜单中，单击 **“所有服务”**。
1. 在服务列表中，单击 **“市场管理”**
1. 在 **“市场管理 - 市场项”** 边栏选项卡上，搜索 **“[smalldisk] Windows Server 2019 数据中心服务器核心 - 自带许可”** 项，并确保其可用。
1. 在与 **AzSHOST-1** 的远程桌面会话中，在浏览器窗口中打开一个新选项卡并导航到 (https://aka.ms/appsvconmasdkfstemplate)。
1. 在 **“AzureStack-QuickStart-Templates / appservice-fileserver-standalone”** 页上，单击 **“azuredeploy.json”**，然后单击 **“原始”**。
1. 选择该页的全部内容并将其复制到剪贴板。
1. 切换回 Azure Stack 管理员门户，并单击 **“+ 创建资源”**。
1. 在 **“新建”** 边栏选项卡上，单击 **“自定义”**，然后单击 **“模板部署”**。
1. 在 **“自定义部署”** 边栏选项卡上，选择 **“在编辑器中生成自己的模板”**。 
1. 在 **“编辑模板”** 边栏选项卡上，将预先创建的模板替换为剪贴板的内容。
1. 在 **“自定义部署”** 边栏选项卡上，单击 **“编辑模板”**。
1. 在 **“编辑模板”** 边栏选项卡的 **“参数”** 部分，设置以下值：

    - **imageReference** 的 defaultValue：设置为 **MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **imageReference** 的 allowedValues：设置为 **MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **fileServerVirtualMachineSize** 的 defaultValue：设置为 **Standard_A1_v2**
    - **fileServerVirtualMachineSize** 的 allowedValues：设置为 **Standard_A1_v2**
    - **adminPassword** 的 defaultValue：设置为 **Pa55w.rd1234**
    - **fileShareOwnerPassword** 的 defaultValue：设置为 **Pa55w.rd1234**
    - **fileShareUserPassword** 的 defaultValue：设置为 **Pa55w.rd1234**

    > **备注：** 这将导致 **“参数”** 部分包含以下内容：

    ```json
      "parameters": {
        "fileServerVirtualMachineSize": {
          "type": "string",
          "defaultValue": "Standard_A1_v2",
          "allowedValues": [
            "Standard_A1_v2",
          ],
          "metadata": {
            "description": "Size of vm"
          }
        },
        "imageReference": {
          "type": "string",
          "defaultValue": "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest",
          "allowedValues": [
            "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest"
          ],
          "metadata": {
            "description": "Please ensure the image is available. publisher: MicrosoftWindowsServer | offer: WindowsServer | sku: 2016-Datacenter"
          }
        },
        "dnsNameForPublicIP": {
          "type": "string",
          "defaultValue": "appservicefileshare",
          "maxLength": 63,
          "metadata": {
            "description": "Unique DNS Name for the Public IP used to access the file share.It must be lowercase. It should match the following regular expression, or it will raise an error: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$"
          }
        },
        "adminUsername": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "File server Admin user"
          }
        },
        "adminPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "File server Admin password"
          }
        },
        "fileShareOwner": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "fileshare owner username"
          }
        },
        "fileShareOwnerPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare owner password"
          }
        },
        "fileShareUser": {
          "type": "string",
          "defaultValue": "fileshareuser",
          "metadata": {
            "description": "fileshare user"
          }
        },
        "fileShareUserPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare user password"
          }
        },
        "vmExtensionScriptLocation": {
          "type": "string",
          "defaultValue": "https://raw.githubusercontent.com/Azure/azurestack-quickstart-templates/master/appservice-fileserver-standalone",
          "metadata": {
            "description": "File Server extension script Url"
          }
        }
      },
    ```

1. 在 **“编辑模板”** 边栏选项卡的 **“变量”** 部分中，将 **“sku: 2016-Datacenter”** 替换为 **“sku: 2019-Datacenter-Core-smalldisk”**，并选择 **“保存”**。
1. 回到 **“自定义部署”** 边栏选项卡，在 **“订阅”** 下拉列表中，选择 **“默认提供商订阅”**，然后在 **“资源组”** 部分中选择 **“sql.resources-RG”**。
1. 在 **“自定义部署”** 边栏选项卡上，单击 **“查看 + 创建”**，然后单击 **“创建”**。

    > **备注：** 等待部署完成。该操作需约 15 分钟。


#### 任务 3：安装应用服务资源提供程序

在此任务中，你将：

- 安装应用服务资源提供程序

1. 在与 **AzSHOST-1** 的远程桌面会话中，在 Azure Stack 管理员门户的中心菜单中，单击 **“所有服务”**。
1. 在服务列表中，单击 **“市场管理”**。
1. 在 **“市场管理 - 市场项”** 边栏选项卡上，搜索 **“Windows Server 2016 数据中心 - 自带许可”** 项，并确保其可用。
1. 在 **“市场管理 - 市场项”** 边栏选项卡上，搜索 **“自定义脚本扩展”** 项，并确保其可用。
1. 在与 **AzSHOST-1** 的远程桌面会话中，启动 Web 浏览器并导航到 (https://aka.ms/appsvconmasinstaller) 以下载 **AppService.exe**，下载完成后，将该文件复制到 **C:\\Downloads\\AppServiceRP** 文件夹（根据需要创建该文件夹）。
1. 在 Web 浏览器中，导航到 (https://aka.ms/appsvconmashelpers) 以下载 **AppServiceHelperScripts.zip**，下载完成后，将其内容解压缩到 **C:\\Downloads\\AppServiceRP** 文件夹。
1. 在与 **AzSHOST-1** 的远程桌面会话中，以管理员身份启动 Windows PowerShell。
1. 从 **“Administrator: Windows PowerShell”** 窗口中运行以下命令，以在 Azure Stack 上创建应用服务所需的证书：

    ```powershell
    Set-Location -Path C:\Downloads\AppServiceRP
    Get-ChildItem -Path '.\' -File -Recurse | Unblock-File

    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-AppServiceCerts.ps1 `
	-pfxPassword $pfxPass `
	-DomainName 'local.azurestack.external'
    ```

1. 从 **“Administrator: Windows PowerShell”** 窗口中运行以下命令，为安装应用服务提供程序所需的 Azure Stack Hub 创建 Azure 资源管理器根证书：

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'

    # 添加特权终结点访问所需的 cloudadmin 凭据。
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object System.Management.Automation.PSCredential ("CloudAdmin@$domain", $cloudAdminPass)

    .\Get-AzureStackRootCert.ps1 -PrivilegedEndpoint $privilegedEndpoint -CloudAdminCredential $cloudAdminCreds
    ```

    > **备注：** 该脚本在本地文件夹中创建一个名为 AzureStackCertificationAuthority.cer 的文件，其中包含 Azure Stack Hub 的 Azure 资源管理器根证书。

1. 从 **“Administrator:** Windows PowerShell”窗口中运行以下命令，以创建安装应用服务提供程序所需的 AD FS 应用：

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $adminArmEndpoint = 'adminmanagement.local.azurestack.external'
    $certificateFilePath = 'C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx'
    $certificatePassword = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-ADFSIdentityApp.ps1 `
       -AdminArmEndpoint $adminArmEndpoint `
       -PrivilegedEndpoint $privilegedEndpoint `
       -CloudAdminCredential $cloudAdminCreds `
       -CertificateFilePath $certificateFilePath `
       -CertificatePassword $certificatePassword
    ```

1. 在脚本的输出中，复制表示生成的 AD FS 应用程序 ID 的 GUID。 

    > **备注：** 请确保记录此 GUID。你将在此任务的后面部分使用它。

1. 从 **“Administrator: Windows PowerShell”** 窗口以及从 **“Administrator: Windows PowerShell”** 窗口，运行以下命令以启动 **AppService.exe**：

    ```
    .\AppService.exe
    ```

    > **备注：** 这将启动 Microsoft Azure 应用服务安装向导。

1. 单击 **“部署应用服务或将其升级为最新版本”**。
1. 在 **“MICROSOFT 软件补充许可条款”** 页上，查看内容，单击 **“我已阅读、理解并同意这些许可条款”** 复选框，然后单击 **“下一步”**。
1. 在显示第三方许可条款的页面上，查看内容，单击 **“我已阅读、理解并同意这些许可条款”** 复选框，然后单击 **“下一步”**。
1. 在显示管理员终结点和租户 ARM 终结点的页面上，验证信息是否正确，然后单击 **“下一步”**。
1. 在“Azure Stack 应用服务云信息”页上，确保选择了 **“凭证”** 选项，然后单击 **“连接”**。
1. 出现提示时，使用 **CloudAdmin@AzureStack.local** 和密码 **Pa55w.rd1234** 登录。
1. 回到“Azure Stack 应用服务云信息”页，在 **“Azure Stack 订阅”** 下拉列表中，选择 **“默认提供商订阅”**，在 **“Azure Stack 位置”** 下拉列表中，选择 **“本地”**，然后单击 **“下一步”**。
1. 在 **“虚拟网络配置”** 中，接受默认设置并单击 **“下一步”**。
1. 在下一页上，指定以下信息并单击 **“下一步”**：

    - 文件共享 UNC 路径： **\\appservicefileshare.local.cloudapp.azurestack.external\websites**
    - 文件共享所有者： **fileshareowner**
    - 文件共享所有者密码： **Pa55w.rd1234**
    - 文件共享用户： **fileshareuser**
    - 文件共享用户密码：**Pa55w.rd1234**

1. 在下一页上，指定标识你在此任务的前面部分生成的应用程序 ID 的设置，然后单击 **“下一步”**：

    - 标识应用程序 ID：你在此任务的前面部分复制的 GUID
    - 标识应用程序证书文件 (*.pfx)： **C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx**
    - 标识应用程序证书文件 (*.pfx) 密码：**Pa55w.rd1234pfx**
    - Azure 资源管理器 (ARM) 根证书文件 (*.cer)： **C:\Downloads\AppServiceRP\AzureStackCertificationAuthority.cer**

1. 在下一页上，指定标识证书文件及其相应密码的设置：

    - 应用服务默认 SSL 证书文件 (*.pfx)： **C:\Downloads\AppServiceRP\_.appservice.local.azurestack.external.pfx**
    - 应用服务默认 SSL 证书 (*.pfx) 密码：**Pa55w.rd1234pfx**
    - 应用服务 API SSL 证书文件 (*.pfx)： **C:\Downloads\AppServiceRP\api.appservice.local.azurestack.external.pfx**
    - 应用服务 API SSL 证书 (*.pfx) 密码： **Pa55w.rd1234pfx**
    - 应用服务发布服务器证书文件 (*.pfx)： **C:\Downloads\AppServiceRP\ftp.appservice.local.azurestack.external.pfx**
    - 应用服务发布服务器 SSL 证书 (*.pfx) 密码： **Pa55w.rd1234pfx**

1. 在下一页上，指定 SQL Server 设置：

    - SQL Server 名称： **sqlhost1.local.cloudapp.azurestack.external**
    - SQL sysadmin 登录名： **SQLAdmin**
    - SQL sysadmin 密码： **Pa55w.rd1234**

1. 在下一页上，指定应用服务部署实例的数目和 SKU：

    - 控制器角色： **2 个实例 - Standard_A1_v2 - [1 个核心，2048 MB]**
    - 管理角色： **1 个实例 - Standard_A2_v2 - [2 个核心，4096 MB]**
    - 发布服务器角色： **1 个实例 - Standard_A1_v2 - [1 个核心，2048 MB]**
    - 前端角色： **1 个实例 - Standard_A1_v2 - [1 个核心，2048 MB]**
    - 共享辅助角色： **1 个实例 - Standard_A1_v2 - [1 个核心，2048 MB]**

1. 单击 **“下一步”**。
1. 在下一页上的 **“选择平台映像”** 下拉列表中，选择 **“2016 Datacenter - latest”** 映像，然后单击 **“下一步”**。
1. 在下一页上，为部署指定以下管理员凭据：

    - 辅助角色虚拟机管理员： **SAWorkerAdmin**
    - 辅助角色虚拟机密码：**Pa55w.rd1234**
    - 确认密码：**Pa55w.rd1234**
    - 其他角色虚拟机管理员： **SAORoleAdmin**
    - 其他角色虚拟机密码： **Pa55w.rd1234**
    - 确认密码： **Pa55w.rd1234**

1. 单击 **“下一步”**。
1. 在“摘要”页上，单击 **“选择并单击‘下一步’以开始部署”** 复选框，然后单击 **“下一步”** 开始部署。

    > **备注：** 等待安装完成。此过程可能需要 2-3 小时。

1. 安装完成后，单击 **“退出”**。


#### 任务 4：验证应用服务资源提供程序的安装

在此任务中，你将：

- 验证应用服务资源提供程序的安装

1. 在与 **AzSHOST-1** 的远程桌面会话中，在显示 Azure Stack 管理员门户的 Web 浏览器的中心菜单中，选择 **“所有服务”**，在 **“所有服务”** 边栏选项卡上，选择 **“管理”**，然后在服务列表中，单击 **“应用服务”**。 

    > **备注：** **“应用服务”** 条目可能需要在刷新浏览器页面后才能变得可用。

1. 在 **“应用服务”** 边栏选项卡的 **“Essentials”** 部分中，验证 **“所有角色已就绪”** 消息是否显示在 **“状态”** 标签的下方。

    > **备注：** 等待成功启动所有角色。该操作可能需要额外的 15-20 分钟。

>**回顾**： 在本练习中，你在 Azure Stack Hub 上安装了应用服务资源提供程序。


### 练习 2：探索 Azure Stack Hub 上应用服务资源提供程序的管理任务

在本练习中，你将探索 Azure Stack Hub 上应用服务资源提供程序的管理任务。

1. 探索应用服务资源的缩放功能
1. 探索应用服务资源提供程序的备份设置


### 任务 1：探索应用服务资源的缩放功能

在此任务中，你将：

- 查看应用服务资源的缩放功能
- 查看应用服务资源提供程序的备份设置

1. 在与 **AzSHOST-1** 的远程桌面会话中，在显示 Azure Stack 管理员门户的 Web 浏览器中的 **“应用服务”** 边栏选项卡上，单击 **“角色”**。
1. 在 **“应用服务” | “角色”** 边栏选项卡上，查看角色列表和相应实例。
1. 在 **“应用服务” | “角色”** 边栏选项卡的 **“控制器”** 行中，单击右侧的省略号，然后在下拉列表中，留意 **“虚拟机”** 条目。

    > **备注：** 控制器角色通过使用虚拟机实现，因此在安装应用服务资源提供程序期间会选择控制器的 2 个实例。

1. 在 **“应用服务” | “角色”** 边栏选项卡的其余行中，单击右侧的省略号，然后在下拉列表中，留意 **“ScaleSet”** 条目。

    > **备注：** 所有其他角色均通过使用规模集实现，以实现缩放。

1. 在 **“应用服务” | “角色”** 边栏选项卡上，请留意当前仅具有 **“共享”** 辅助角色层级中的 Web 辅助角色。 
1. 在 **“应用服务” | “角色”** 边栏选项卡上，在左侧的垂直菜单中，选择 **“辅助角色层级”**。
1. 在 **“应用服务” | “辅助角色层级”** 边栏选项卡上，单击 **“+ 添加”**。 
1. 在 **“创建”** 边栏选项卡上，查看可用选项，包括允许在 **“共享”** 和 **“专用”** 之间进行选择的 **“计算模式”** 下拉列表。

    > **备注：** 可以使用自定义软件将各种大小的虚拟机部署为所选辅助角色层级中的虚拟机。

1. 关闭 **“创建”** 边栏选项卡，而不进行任何更改。

    > **备注：** 预配过程可能需要一个多小时。


### 任务 2：探索应用服务资源提供程序的备份设置

在此任务中，你将：

- 查看应用服务资源提供程序的备份设置。

> **备注：** Azure Stack Hub 上的应用服务备份包含以下主要组件

- 资源提供程序基础结构
- 资源提供程序机密 
- 托管计量数据库的 SQL Server 实例
- 存储在应用服务文件共享中的用户工作负载内容

    > **备注：** 可以在恢复期间使用应用服务恢复 PowerShell cmdlet 基于备份重新创建资源提供程序基础结构配置。有关恢复过程的详细信息，请参阅 [Azure Stack Hub 上的应用服务恢复](https://docs.microsoft.com/zh-cn/azure-stack/operator/app-service-recover?view=azs-2008)。

1. 在与 **AzSHOST-1** 的远程桌面会话中，在显示 Azure Stack 管理员门户的 Web 浏览器的 **“应用服务”** 边栏选项卡上，单击 **“机密”**。
1. 在 **“应用服务”\| “机密”** 边栏选项卡上，单击 **“下载机密”**，然后单击 **“保存”**。
1. 验证 **SystemSecrets.json** 文件是否已下载到 **AzSHOST-1** 上的 **Downloads** 文件夹。

    > **备注：** 应将 **SystemSecrets.json** 文件复制到一个安全位置，并在每次轮换机密时重复此过程。 

    > **备注：** 虽然备份 **Appservice_hosting** 和 **Appservice_metering** 数据库的建议方法包含使用 Azure 备份服务器的 SQL Server 维护计划，但你也可以使用 SQL Server PowerShell 模块 cmdlet 进行备份。

1. 在与 **AzSHOST-1** 的远程桌面会话中，切换到与 **sqlhost1.local.cloudapp.azurestack.external** 的远程桌面会话。 
1. 在与 **SqlHOST1** 的远程桌面会话中，以管理员身份启动 Windows PowerShell。
1. 在与 **SqlHOST1** 的远程桌面会话中，从 **“Administrator: Windows PowerShell”** 提示符中运行以下命令，以执行应用服务数据库的本地备份：

    ```powershell
    $date = Get-Date -Format 'yyyyMMdd'
    New-Item -ItemType Directory -Path 'C:\Backups'
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_hosting' -BackupFile "C:\Backups\appservice_hosting_$date.bak" -CopyOnly
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_metering' -BackupFile "C:\Backups\appservice_metering_$date.bak" -CopyOnly
    ```

    > **备注：** 应用服务在其指定文件共享上存储租户应用信息。虽然备份文件共享的建议方法包含使用 Azure 备份服务器，但你也可以使用任何文件复制实用工具来进行备份。

1. 切换到与 **AzSHOST-1** 的远程桌面会话，在与 **AzSHOST-1** 的远程桌面会话中，在显示 Azure Stack 管理员门户的 Web 浏览器的 **“应用服务”** 边栏选项卡上，单击 **“系统配置”**。
1. 在显示 Azure Stack 管理员门户的 Web 浏览器中，在 **“应用服务” \| “系统配置”** 边栏选项卡上，留意文件共享的完整路径 (**\\\\appservicefileshare.local.cloudapp.azurestack.external\\websites**)
1. 切换到与 **AzSHOST-1** 的远程桌面会话，从 **“Administrator: Windows PowerShell”** 窗口中运行以下命令，将应用服务文件共享的内容复制到本地文件系统：

    ```powershell
    $source = '\\appservicefileshare.local.cloudapp.azurestack.external\websites'
    $date = Get-Date -Format 'yyyyMMdd'
    $destination = "C:\Backups\FileShare\$date"
    New-Item -ItemType Directory -Path $destination -Force

    $fileshareusername = 'fileshareowner'
    $fileshareuserpassword = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $fileshareusercreds = New-Object System.Management.Automation.PSCredential ($fileshareusername, $fileshareuserpassword)
    New-PSDrive -Name 'S' -Root $source -PSProvider 'FileSystem' -Credential $fileshareusercreds
    robocopy $source $destination /e /r:1 /w:1
    Remove-PSDrive -Name 'S'
    ```

>**回顾**： 在本练习中，你探索了 Azure Stack Hub 上应用服务资源提供程序的管理任务。


### 练习 3：在 Azure Stack Hub 中交付应用服务资源

在本练习中，你将向 Azure Stack Hub 用户交付应用服务资源。

1. 向用户提供应用服务资源（以云操作员的身份）
1. 创建 Web 应用（以用户身份）


### 任务 1：向用户提供应用服务资源（以云操作员的身份）

在此任务中，你将：

- 向用户提供应用服务资源（以云操作员的身份）

1. 在与 **AzS-HOST1** 的远程桌面会话中，切换到显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的浏览器窗口。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“+ 创建资源”**。
1. 在 **“新建”** 边栏选项卡上，单击 **“套餐 + 计划”**，然后单击 **“计划”**。
1. 在 **“新建计划”** 边栏选项卡的 **“基本信息”** 选项卡上，指定以下设置：

    - 显示名称： **app-service-plan1**
    - 资源名称： **app-service-plan1**
    - 资源组：新资源组名称 **app-service-plans-RG**

1. 单击 **“下一步: 服务 >”**。
1. 在 **“新建计划”** 边栏选项卡的 **“服务”** 选项卡上，选中 **“Microsoft.Web”** 复选框。
1. 单击 **“下一步: 配额>”**。
1. 在 **“新建计划”** 边栏选项卡的 **“配额”** 选项卡上，选择 **“新建”**，然后在 **“创建”** 边栏选项卡上，指定以下设置，并单击 **“确定”**：

    - 名称： **app-service-quota1**
    - 应用服务计划： **自定义** **20**
    - 共享应用服务计划： **自定义** **10**
    - 专用应用服务计划： **自定义 10**
    - 定价 SKU： **所选** **自定义** **2** （**“免费”** 和 **“共享”**）
    - 消耗计划： **已启用**

    >**备注**： 若要在消耗计划模型中提供 Azure Functions，需要部署共享的 Web 辅助角色。

1. 回到 **“新建计划”** 边栏选项卡的 **“配额”** 选项卡，单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，返回到 **“新建”** 边栏选项卡，单击 **“套餐”**。
1. 在 **“新建套餐”** 边栏选项卡的 **“基本信息”** 选项卡中，指定以下设置：

    - 显示名称： **app-service-offer1**
    - 资源名称： **app-service-offer1**
    - 资源组： **app-service-offers-RG**
    - 公开提供此套餐： **是**

1. 单击 **“下一步: 基本计划 >”**。 
1. 在 **“新建套餐”** 边栏选项卡的 **“基本计划”** 选项卡上，选中 **“app-service-plan1”** 条目旁边的复选框。
1. 单击 **“下一步:  附加产品计划 >”**。
1. 保留 **“附加产品计划”** 设置为默认值，单击 **“查看 + 创建”**，然后单击 **“创建”**。

    >**备注**： 等待部署完成。这应该只需要几秒钟时间。


#### 任务 2：创建 Web 应用（以用户身份）

在此任务中，你将：

- 创建测试用户帐户
- 创建 Web 应用（以用户身份）

1. 在与 **AzS-HOST1** 的远程桌面会话中，单击 **“开始”**，在 **“开始”** 菜单中，单击 **“Windows 管理工具”**，然后在管理工具列表中，双击 **“Active Directory 管理中心”**。
1. 在 **“Active Directory 管理中心”** 控制台中，单击 **“azurestack(本地)”**。
1. 在“详细信息”窗格中，双击 **“用户”** 容器。
1. 在 **“任务”** 窗格的 **“用户”** 部分中，单击 **“新建” -> “用户”**。
1. 在 **“创建用户”** 窗口中，指定以下设置，然后单击 **“确定”**： 

    - 全名： **T1U1**
    - 用户 UPN 登录名： **t1u1@azurestack.local**
    - 用户 SamAccountName： **azurestack\t1u1**
    - 密码： **Pa55w.rd**
    - 密码选项： **“其他密码选项” -> “密码永不过期”**

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 Web 浏览器的 InPrivate 会话。
1. 在 Web 浏览器窗口中，导航到 [Azure Stack Hub 用户门户](https://portal.local.azurestack.external)，并使用 **t1u1@azurestack.local** 和密码 **Pa55w.rd** 登录。
1. 在 Azure Stack Hub 用户门户中，单击“仪表板”上的 **“获取订阅”** 磁贴。
1. 在 **“获取订阅”** 边栏选项卡的 **“名称”** 文本框中，键入 **“t1u1-app-service-subscription1”**。
1. 在套餐列表中，选择 **“app-service-offer1”**，然后单击 **“创建”**。
1. 出现消息 **“订阅已创建。 必须刷新门户才能开始使用订阅”** 时，请单击 **“刷新”**。 
1. 在 Azure Stack Hub 租户门户中心菜单中，单击 **“+ 创建资源”**。
1. 在服务列表中，单击 **“Web + 移动”**，然后单击 **“Web 应用”**。 
1. 在 **“Web 应用”** 边栏选项卡上，指定以下设置：

    - 订阅： **t1u1-app-service-subscription1**
    - 应用名称： **t1u1webapp1**
    - 资源组：新资源组名称 **webapps-RG**

1. 在 **“Web 应用”** 边栏选项卡上，单击 **“应用服务计划/位置”**，在 **“应用服务计划”** 边栏选项卡上，单击 **“+ 新建”**。 
1. 在 **“新建应用服务计划”** 边栏选项卡上，指定以下设置：

    - 应用服务计划： **appserviceplan1**
    - 位置： **本地**

1. 在 **“新建应用服务计划”** 边栏选项卡上，单击 **“定价层”**。
1. 在 **“规格选取器”** 边栏选项卡上，选择 **“F1”** 定价层并单击 **“应用”**。
1. 返回“新建应用服务计划”边栏选项卡，单击 **“确定”**。
1. 返回 **“Web 应用”** 边栏选项卡，单击 **“创建”**。

    >**备注**： 等待部署完成。所需时间应该不超过一分钟。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 Azure Stack Hub 用户门户的 Web 浏览器的 InPrivate 会话中，在中心菜单中选择 **“所有资源”**。
1. 在 **“所有资源”** 边栏选项卡的订阅筛选器下拉列表中，选择 **“t1u1-app-service-subscription1”** 条目，然后单击 **“刷新”**。
1. 在 **“所有资源”** 边栏选项卡上的资源列表中，单击 **“t1u1webapp1”** 条目。
1. 在 **“t1u1webapp1”** 边栏选项卡上，单击 **“浏览”**。

    >**备注**： 此操作应会打开另一个浏览器选项卡，其中显示新预配的 Web 应用的默认主页。

>**回顾**： 在本练习中，你向用户提供了应用服务，并以租户用户身份创建了 Web 应用。