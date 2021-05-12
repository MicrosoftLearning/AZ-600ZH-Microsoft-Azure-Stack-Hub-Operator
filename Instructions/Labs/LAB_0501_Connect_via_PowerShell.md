---
lab:
    title: '实验室：通过 PowerShell 连接到 Azure Stack Hub'
    module: '模块 5：管理基础结构'
---

# 实验室 - 通过 PowerShell 连接到 Azure Stack Hub
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要能够使用 PowerShell 管理环境。你还需能够偶尔以用户身份通过 PowerShell 连接到 Azure Stack Hub。 

## 目标

在本实验室中，你将能够：

- 通过 PowerShell 连接到 ASDK 操作员和用户环境

## 实验室环境

本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

>**备注**： 有关连接到与 Azure Active Directory (Azure AD) 集成的 Azure Stack Hub 的信息，请参阅[使用 PowerShell 连接到 Azure Stack Hub](https://docs.microsoft.com/zh-cn/azure-stack/operator/azure-stack-powershell-configure-admin?view=azs-2008&tabs=az1%2Caz2%2Caz3#connect-with-azure-ad)。

实验室环境具有以下配置：

- 在具有以下接入点的 **AzS-HOST1** 服务器上运行的 ASDK 部署：

  - 管理员门户：https://adminportal.local.azurestack.external
  - 管理员 ARM 终结点：https://adminmanagement.local.azurestack.external
  - 用户门户：https://portal.local.azurestack.external
  - 用户 ARM 终结点：https://management.local.azurestack.external

- 管理用户：

  - ASDK 云操作员用户名：**CloudAdmin@azurestack.local**
  - ASDK 云操作员密码：**Pa55w.rd1234**
  - ASDK 主机管理员用户名：**AzureStackAdmin@azurestack.local**
  - ASDK 主机管理员密码：**Pa55w.rd1234**

在本实验室课程中，你将安装通过 PowerShell 管理 Azure Stack Hub 所需的软件。 

## 说明

### 练习 1：通过 Azure PowerShell 连接到 ASDK

在本练习中，你将通过 PowerShell 从 ASDK 主机连接到 ASDK 的 Admin ARM 终结点：

1. 安装与 Azure Stack Hub 兼容的 Az PowerShell 模块
1. 下载 Azure Stack Hub 工具。
1. 通过 PowerShell 配置并连接到 Azure Stack Hub 操作员环境
1. 通过 PowerShell 配置并连接到 Azure Stack Hub 用户环境

#### 任务 1：安装与 Azure Stack Hub 兼容的 Azure PowerShell 模块

在此任务中，你将：

- 删除所有的现有 Azure 和 Az PowerShell 模块。
- 安装与 Azure Stack Hub 兼容的 Az PowerShell 模块并为其配置先决条件。
- 安装与 Azure Stack Hub 兼容的 Az PowerShell 模块。

>**备注**： Azure 资源管理器 (AzureRM) PowerShell 模块的所有版本均已过时，但并没有停止提供支持。不过，在与 Azure 和 Azure Stack Hub 交互时，建议使用的 PowerShell 模块是 Az PowerShell 模块。 

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell**。
1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，以删除所有现有版本的 Azure PowerShell 和 Az PowerShell 模块：

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```

    >**备注**： 如果收到有关正在使用的模块的错误消息，请关闭 Windows PowerShell 会话再重新打开，然后重新运行上述命令。

1. 从 **“Administrator: Windows PowerShell”** 提示符运行以下命令，从 **C:\\Program Files\\WindowsPowerShell\\Modules** 和 **C:\\Users\\AzureStackAdmin\\Documents\\PowerShell\\Modules\\** 文件夹中删除所有以 **Azure**、 **Az** 或 **Azs** 开头的文件夹。

    ```powershell
    Get-ChildItem -Path 'C:\Program Files\WindowsPowerShell\Modules' -Include 'Az*' -Recurse -Force | Remove-Item -Force -Recurse
    Get-ChildItem -Path 'C:\Users\AzureStackAdmin\Documents\PowerShell\Modules' -Include 'Az*' -Recurse -Force | Remove-Item -Force -Recurse
    ```

    >**备注**： 如果收到有关正在使用的模块的错误消息，请关闭 Windows PowerShell 会话再重新打开，然后重新运行上述命令。


1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 Microsoft Edge，导航到 [PowerShell 版本页面](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)。 
1. 从 [PowerShell 版本页面](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)，下载并安装最新版本的 PowerShell。 
1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell 7。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，将 PowerShell 库配置为受信任的存储库

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以安装 PowerShellGet：

    ```powershell
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

    >**备注**： 忽略有关正在使用的模块的任何警告消息。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，为 Azure Stack Hub 安装 PowerShell Az 模块：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**)备注**： 忽略有关已可用命令的任何错误消息。


#### 任务 2：下载 Azure Stack Hub 工具 

在此任务中，你将：

- 从 GitHub 下载 Azure Stack Hub 工具

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以下载 Azure Stack Tools：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**备注**： 此步骤将包含托管 Azure Stack Hub 工具的 GitHub 存储库的存档复制到本地计算机，并将存档扩展到 **C:\\AzureStack-Tools-master** 文件夹。这些工具包含 PowerShell 模块，这些模块提供了一系列功能，包括识别 Azure Stack Hub 功能、管理 Azure Stack Hub VM 基础结构和映像、配置资源管理器策略、向 Azure 注册 Azure Stack Hub、Azure Stack Hub 部署、与 Azure Stack Hub 建立连接、Azure Stack Hub 租户管理以及验证 Azure Stack Hub 资源管理器模板。 


#### 任务 3：通过 PowerShell 配置并连接到 Azure Stack Hub 操作员环境

在此任务中，你将：

- 通过 PowerShell 配置 Azure Stack Hub 操作员环境
- 通过 PowerShell 连接到 Azure Stack Hub 操作员环境
- 通过 PowerShell 验证与 Azure Stack Hub 操作员环境的连接

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以注册 Azure Stack Hub 操作员 PowerShell 环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

    >**备注**： 验证命令是否返回以下输出：

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackAdmin https://adminmanagement.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以使用 AzureStack\CloudAdmin 凭据登录到 Azure Stack Hub 操作员 PowerShell 环境：

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin' -UseDeviceAuthentication
    ```

1. 在 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口中，查看生成的消息，打开 Web 浏览器窗口，导航到 [adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth) 页面，键入已查看的消息中包含的代码，然后单击 **“继续”**。 

1. 出现提示时，请使用以下凭据登录：

    - 用户名： **CloudAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 切换回 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，并验证是否已成功使用 **CloudAdmin@azurestack.local** 进行身份验证。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以列出 Azure Stack Hub 管理员订阅

    ```powershell
    Get-AzSubscription
    ```

    >**备注**： 确认输出包括 **“默认提供商订阅”**、 **“计量订阅”** 和 **“消耗订阅”**。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以验证相应的 PowerShell 环境上下文：

    ```powershell
    Get-AzContext
    ```


#### 任务 4：通过 PowerShell 配置并连接到 Azure Stack Hub 用户环境

在此任务中，你将：

- 通过 PowerShell 配置 Azure Stack Hub 用户环境
- 通过 PowerShell 连接到 Azure Stack Hub 用户环境
- 通过 PowerShell 验证与 Azure Stack Hub 用户环境的连接

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 PowerShell 7。
1. 从 **“C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以注册面向 Azure Stack Hub 用户环境的 Azure 资源管理器环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

    >**备注**： 验证命令是否返回以下输出：

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackUser https://management.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. 从 **“C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以登录到 Azure Stack Hub PowerShell 环境。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser'
    ```

    >**备注**： 这将自动打开 Web 浏览器窗口，提示你进行身份验证。

1. 出现提示时，请使用以下凭据登录：

    - 用户名： **CloudAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，以列出 Azure Stack Hub 管理员订阅

    ```powershell
    Get-AzSubscription
    ```

    >**备注**： 确认输出不包括 **“默认提供商订阅”**、 **“计量订阅”** 和 **“消耗订阅”**。

>**回顾**： 在本练习中，你通过 PowerShell 连接了 Azure Stack Hub 操作员环境和用户环境。
