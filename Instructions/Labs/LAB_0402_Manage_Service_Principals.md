---
lab:
    title: '实验室：在 Azure Stack Hub 中管理服务主体'
    module: '模块 4：管理标识和访问'
---

# 实验室 - 在 Azure Stack Hub 中管理服务主体
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你计划使用内部开发的应用程序来管理 Azure Stack Hub。若要允许应用程序进行身份验证，需创建服务主体，并为其分配默认提供商订阅中的参与者角色。

>**备注**： 需要通过 Azure 资源管理器部署或配置资源的应用程序必须由其自身的标识表示。正如用户由称为用户主体的安全主体表示一样，应用由服务主体表示。服务主体为应用提供标识，以便向该应用仅委派必需的权限。

应用必须在身份验证期间出示凭据。此身份验证由两个元素组成：

- 应用程序 ID，有时也称为客户端 ID。唯一标识 Active Directory 租户中的应用注册的 GUID。
- 与应用程序 ID 关联的机密。你可以生成一个客户端密码字符串（相当于密码），也可以指定一个 X509 证书。

在本实验室中，你将使用机密。有关使用证书进行身份验证的详细信息，请参阅[使用应用标识访问 Azure Stack Hub 资源](https://docs.microsoft.com/zh-cn/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-disconnected)。

## 目标

完成本实验室后，你将能够：

- 在 Azure Stack Hub AD FS 集成方案中创建和管理服务主体。

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

本实验室中所述的方法特定于 AD FS 集成方案。要了解在使用 Azure Active Directory (Azure AD) 作为标识提供程序时如何实现基于服务主体的身份验证，请参阅[使用应用标识访问 Azure Stack Hub 资源](https://docs.microsoft.com/zh-cn/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-connected)。


### 练习 1：在 Azure Stack Hub 中创建和配置服务主体

在本练习中，你将建立与特权终结点的 PowerShell 远程会话以创建服务主体，并使用 Azure Stack Hub 管理员门户为其分配参与者角色。练习包含以下任务：

1. 创建服务主体
1. 向服务主体分配参与者角色


#### 任务 1：创建服务主体

在此任务中，你将：

- 通过 PowerShell 连接到特权终结点，并创建服务主体。

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell 7。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以建立与特权终结点的 PowerShell 远程处理会话：

    ```powershell
    $session = New-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以创建新的应用注册（和服务主体对象），并将其引用存储在 $spObject 变量中：

    ```powershell
    $spObject = Invoke-Command -Session $session -ScriptBlock {New-GraphApplication -Name 'azsmgmt-app1' -GenerateClientSecret}
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以检索 Azure Stack Hub 印花信息并将其引用存储在 $azureStackInfo 变量中：

    ```powershell
    $azureStackInfo = Invoke-Command -Session $session -ScriptBlock {Get-AzureStackStampInformation}
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以终止与特权终结点的 PowerShell 远程处理会话：

    ```powershell
    $session | Remove-PSSession
    ```

    >**备注**： 一般情况下，应使用 **Close-PrivilegedEndpoint** cmdlet 关闭特权终结点会话。为了简单起见，我们没有遵循这种做法，这样就无需设置文件共享来托管会话脚本日志。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以使用你之前在此任务中检索到的 Azure Stack Hub 印花信息来设置将用于配置服务主体的变量的值，这些值分别引用用于 Azure 资源管理器用户操作的 Azure Stack Hub 终结点、获取用于访问图形 API 的 Oauth 令牌的受众和标识提供者的 GUID：

    ```powershell
    $armUseEndpoint = $azureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以注册和设置 Azure Stack Hub 用户环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint $armUseEndpoint
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以服务主体身份登录到 AzureStackUser 环境：

    ```powershell
    $securePassword = $spObject.ClientSecret | ConvertTo-SecureString -AsPlainText -Force
    $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $spObject.ClientId, $securePassword
    $spUserSignIn = Connect-AzAccount -Environment 'AzureStackUser' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以验证登录是否成功：

    ```powershell
    $spUserSignIn
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以删除当前身份验证上下文：

    ```powershell
    Remove-AzAccount -Username $credential.UserName
    ```

    >**备注**： 现在，你将重复同等的步骤顺序，验证是否可以对 Azure Stack Hub 管理员环境进行身份验证：

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以使用你之前在此任务中检索到的 Azure Stack Hub 印花信息来设置将用于配置服务主体的变量的值，这些值分别引用用于 Azure 资源管理器管理操作的 Azure Stack Hub 终结点、获取用于访问图形 API 的 Oauth 令牌的受众和标识提供者的 GUID：

    ```powershell
    $armAdminEndpoint = $azureStackInfo.AdminExternalEndpoints.AdminResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以注册和设置 Azure Stack Hub 管理员环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint $armAdminEndpoint
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以服务主体身份登录到 AzureStackAdmin 环境：

    ```powershell
    $spAdminSignIn = Connect-AzAccount -Environment 'AzureStackAdmin' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以验证登录是否成功：

    ```powershell
    $spAdminSignIn
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以显示新服务主体的属性：

    ```powershell
    $spObject
    ```

    >**备注**： 输出格式应如下所示：

    ```
    ApplicationIdentifier : S-1-5-21-2657257302-3827180852-1812683747-1510
    ClientId              : 6a3fb4ad-838a-47b1-a93c-f3e4a1b683c8
    Thumbprint            :
    ApplicationName       : Azurestack-azsmgmt-app2-53508ec5-d7d9-4761-876a-3602542c2965
    ClientSecret          : 3fKPtUg37YraCk1IaFtdqeyTpVplXDqc25Dj3bUs
    PSComputerName        : AzS-ERCS01
    RunspaceId            : 6b142339-b67f-490e-a258-40983c0cd8ea
    ```

    >**备注**： 记录 **ApplicationName** 属性的值。在下一个任务中需要使用它。此外，你应该记录 **ClientSecret** 属性的值，并将其提供给实现管理应用程序的开发人员。


#### 任务 2：向服务主体分配参与者角色

在此任务中，你将：

- 使用 Azure Stack Hub 管理员门户将参与者角色分配给服务主体。

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器的中心菜单中，选择 **“所有服务”**。 
1. 在 **“所有服务”** 边栏选项卡上，选择 **“常规”**，然后在服务列表中选择 **“订阅”**。
1. 在 **“订阅”** 边栏选项卡上，选择 **“默认提供商订阅”**。
1. 在 **“默认提供商订阅”** 边栏选项卡上，选择 **“访问控制(IAM)”**。
1. 在 **“默认提供商订阅 | 访问控制(IAM)”** 边栏选项卡上，单击 **“+ 添加”**，然后在下拉菜单中单击 **“添加角色分配”**。
1. 在 **“添加角色分配”** 边栏选项卡上，指定以下设置并单击 **“保存”**：

    - 角色： **参与者**
    - 访问权限分配对象： **Azure AD 用户、组或服务主体**
    - 选择：搜索并选择在上一个任务中识别的服务主体的 **ApplicationName** 属性的值。

1. 验证角色分配是否成功。

>**回顾**： 在本练习中，你创建了服务主体，并为其分配了参与者角色。 
