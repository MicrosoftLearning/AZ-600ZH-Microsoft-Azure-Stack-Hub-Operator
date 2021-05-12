---
lab:
    title: '实验室：使用 Azure 订阅注册 Azure Stack Hub'
    module: '模块 3：实现数据中心集成'
---

# 实验室 - 使用 Azure 订阅注册 Azure Stack Hub
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要使用 Azure 订阅注册它，才能下载 Azure 市场项并设置对 Azure 的数据报告。 

## 目标

完成本实验室后，你将能够：

- 使用 Azure 订阅注册 Azure Stack Hub 

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


### 练习 1：使用 Azure 订阅注册 Azure Stack Hub。

在本练习中，你将使用 Azure 订阅注册 Azure Stack Hub。

1. 注册 Azure Stack Hub 资源提供程序
1. 执行 Azure Stack Hub 注册
1. 验证 Azure Stack Hub 注册

#### 任务 1：注册 Azure Stack Hub 资源提供程序

在此任务中，你将：

- 在 Azure 订阅中注册 Azure Stack Hub 资源提供程序。

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell 7。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，为 Azure Stack Hub 安装 PowerShell Az 模块：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，对将在本实验室中使用的 Azure 订阅进行身份验证。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureCloud'
    ```

1. 出现提示时，使用将在本实验室中使用的 Azure 订阅中具有参与者角色的 Azure Active Directory (Azure AD) 用户的凭据登录。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以注册 Azure Stack 资源提供程序：

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以确定注册是否已完成：

    ```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.AzureStack | Where-Object {$_.RegistrationState -eq 'Registered'}
    ```

    >**备注**： 等待注册完成。重新运行 **Get-AzResourceProvider** cmdlet，以验证注册状态。


#### 任务 2：执行 Azure Stack Hub 注册

在此任务中，你将：

- 执行 Azure Stack Hub 注册。

1. 在与 **AzSHOST-1** 的远程桌面会话中，从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以建立特权终结点 (PEP) 会话：

    ```powershell
    Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，在与特权终结点的 PowerShell 远程处理会话中，运行以下命令以显示 Azure Stack Hub 印花的摘要配置：

    ```powershell
    Get-AzureStackStampInformation
    ```

1. 在上一步中运行的命令的输出中，标识并记录 **CloudId** 属性的值。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，在与特权终结点的 PowerShell 远程处理会话中，运行以下命令退出会话：

    ```powershell
    Exit-PSSession
    ```

    >**备注**： 一般情况下不应使用 Exit_PSSession 终止与特权终结点的会话，而是改用 **Close-PrivilegedEndpoint** cmdlet。为了简单起见，我们没有遵循这种做法，这样就无需设置文件共享来托管会话脚本日志。

    >**备注**： 还可以从 Azure Stack Hub 管理员门户中 **“本地 \| 属性”** 边栏选项卡识别印花 Cloud ID 属性的值。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以注册针对 Azure Stack 实例的 Azure Stack PowerShell 环境（确保将 `[cloud_ID]` 占位符替换为在 **Get-AzureStackStampInformation** cmdlet 的输出中识别的 **CloudId** 属性的值）：

    ```powershell
    Import-Module .\Registration\RegisterWithAzure.psm1
    $RegistrationName = "[cloud_ID]"
    Set-AzsRegistration `
       -PrivilegedEndpointCredential $adminCredentials `
       -PrivilegedEndpoint 'AzS-ERCS01' `
       -BillingModel 'Development' `
       -RegistrationName $RegistrationName `
       -UsageReportingEnabled:$true
    ```

1. 出现提示时，请使用在 Azure 订阅中具有参与者角色的 Azure AD 用户帐户登录。

    > **备注：** 等待注册完成。此注册可能需要约 20 分钟。


#### 任务 3：验证 Azure Stack Hub 注册

在此任务中，你将：

- 验证是否向 Azure 订阅注册了 Azure Stack Hub

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器中，从 **“仪表板”** 边栏选项卡选择 **“区域管理”** 磁贴。
1. 在 **“本地”** 边栏选项卡上，选择 **“属性”**。 
1. 在 **“本地 \| 属性”** 边栏选项卡上，确认 **“注册状态”** 显示为 **“已注册”**。 

    > **备注：** 状态可以是 **“已注册”**、**“未注册”** 或 **“已过期”**。

>**回顾**： 在本练习中，你已向 Azure 订阅注册了 Azure Stack Hub。
