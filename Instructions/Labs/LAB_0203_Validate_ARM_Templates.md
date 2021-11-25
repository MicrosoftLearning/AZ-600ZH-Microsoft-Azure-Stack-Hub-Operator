---
lab:
    title: '实验室：使用 Azure Stack Hub 验证 Azure 资源管理器 (ARM) 模板'
    module: '模块 2：提供服务'
---

# 实验室 - 使用 Azure Stack Hub 验证 Azure 资源管理器 (ARM) 模板
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要使用现有的 Azure 资源管理器 (ARM) 模板来自动部署 Azure Stack Hub 资源。 

## 目标

完成本实验室后，你将能够：

- 验证 Azure Stack Hub 部署的 ARM 模板。

## 实验室环境

本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

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


### 练习 1：使用 Azure Stack Hub 验证 ARM 模板

在本练习中，你将使用 Azure Stack Hub 验证 ARM 模板。

1. 生成云功能文件 
1. 运行成功的模板验证
1. 运行失败的模板验证
1. 修正模板问题


#### 任务 1：生成云功能文件

在此任务中，你将：

- 生成云功能文件

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

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

    >**备注**： 忽略有关已可用命令的任何错误消息。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，注册 Azure Stack Hub 操作员 PowerShell 环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```   

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，登录到新注册的 **AzureStackAdmin** 环境：

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. 如果出现提示，请使用 **CloudAdmin@azurestack.local** 帐户进行身份验证。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，以下载 Azure Stack Tools：

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**备注**： 此步骤将包含托管 Azure Stack Hub 工具的 GitHub 存储库的存档复制到本地计算机，并将存档扩展到 **C:\\AzureStack-Tools-master** 文件夹。 这些工具包含提供了一系列功能的 PowerShell 模块，其中包括对 Azure Stack Hub 资源管理器模板进行验证的功能。 

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，导入 AzureRM.CloudCapabilities PowerShell 模块：

    ```powershell
    Import-Module .\CloudCapabilities\Az.CloudCapabilities.psm1 -Force
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，生成云功能 JSON 文件： 

    ```powershell
    $path = 'C:\Templates'
    New-Item -Path $path -ItemType Directory -Force
    Get-AzCloudCapability -Location 'local' -OutputPath $path\AzureCloudCapabilities.Json -Verbose
    ```

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动文件资源管理器，导航到 **C:\\Templates** 文件夹并验证是否已成功创建 **AzureCloudCapabilities.Json** 文件。

#### 任务 2：运行成功的模板验证

在此任务中，你将：

- 针对 Azure Stack Hub 快速启动模板运行模板验证器

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 Web 浏览器，然后导航到 Azure Stack Hub 快速启动模板存储库[**适用于 AzureStack 的 Windows 上的 MySql Server** 页面](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows)。 
1. 在 **“适用于 AzureStack 的 Windows 上的 MySql Server”** 页面，单击 **azuredeploy.json**。
1. 在 [AzureStack-QuickStart-Templates/mysql-standalone-server-windows/azuredeploy.json](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/mysql-standalone-server-windows/azuredeploy.json) 页面，查看模板的内容。
1. 切换到 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，运行以下命令下载 azuredeploy.json 文件，然后在 C:\\Templates 文件夹中将其另存为名为 **sampletemplate1.json** 的文件。

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/mysql-standalone-server-windows/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate1.json
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，导入 AzureRM.TemplateValidator PowerShell 模块：

    ```powershell
    Set-Location -Path 'C:\AzureStack-Tools-az\TemplateValidator'
    Import-Module .\AzureRM.TemplateValidator.psm1 -Force
    ```
  
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，验证模板：

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate1.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate1validationreport.html `
        -Verbose
    ```

1. 查看验证的输出并确认没有问题。

    >**备注**： 输出格式应如下所示：

    ```
    Validation Summary:
        Passed: 1
        NotSupported: 0
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate1validationreport.html
    ```

#### 任务 3：运行失败的模板验证

在此任务中，你将：

- 针对 Azure 快速启动模板运行模板验证器

1. 在与 **AzS-HOST1** 的远程桌面会话中，从显示 AzureStack 快速启动模板存储库的 Web 浏览器导航到 Azure 快速启动模板存储库 [**Ubuntu VM 上的 MySQL Server 5.6** 页面](https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/mysql/mysql-standalone-server-ubuntu)
1. 在 **“Ubuntu VM 上的 MySQL Server 5.6”** 页面，单击 **azuredeploy.json**。
1. 在 [azure-quickstart-templates/mysql-standalone-server-ubuntu/azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/application-workloads/mysql/mysql-standalone-server-ubuntu/azuredeploy.json) 页面，查看模板的内容。
1. 切换到 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，运行以下命令下载 azuredeploy.json 文件，然后在 C:\\Templates 文件夹中将其另存为名为 **sampletemplate2.json** 的文件。

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/mysql/mysql-standalone-server-ubuntu/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate2.json
    ```

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开 Web 浏览器，导航到[虚拟机](https://docs.microsoft.com/zh-cn/rest/api/compute/virtualmachines)的 REST API 参考，并标识最新版 Azure API（创作此内容时为 **2020-12-01**）。 
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，在记事本中打开 **sampletemplate2.json** 文件。

    ```powershell
    notepad.exe $path\sampletemplate2.json
    ```

1. 在显示 **sampletemplate2.json** 文件的内容的记事本窗口中，在模板的 `resources` 一节中找到表示虚拟机资源的部分，其格式如下：

    ```json
    {
      "apiVersion": "2017-03-30",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

1. 将 `apiVersion` 密钥的值设置为先前在该任务中标识的虚拟机的最新版 Azure REST API，然后保存更改并使文件保持打开状态。

    >**备注**： 更改前，请记录原始值。在下一任务中，你需将其恢复为原始值。

    >**备注**： 假设 REST API 的版本为 **2020-12-01**，则更改应生成以下结果：

    ```json
    {
      "apiVersion": "2020-12-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

    >**备注**： 其中有意引入了 Azure Stack Hub 尚不支持的配置。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，验证最近修改的模板：

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 查看验证的输出并注意是否存在可支持性问题。

    >**备注**： 输出格式应如下所示：

    ```
    Validation Summary:
        Passed: 0
        NotSupported: 1
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html
    ```

1. 打开 **C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html** 文件，然后查看报表。 

    >**备注**： 报表应包含以下格式的条目： **NotSupported: apiversion (Resource type: Microsoft.Compute/virtualMachines)。Not Supported Values - 2020-12-01**。


#### 任务 4：修正模板问题

在此任务中，你将：

- 修正模板问题。

1. 在与 **AzS-HOST1** 的远程桌面会话中，通过文件资源管理器，导航到 **C:\\Templates** 文件夹并打开 **AzureCloudCapabilities.json** 文件。
1. 在 **AzureCloudCapabilities.json** 文件中，找到 `"ResourceTypeName": "virtualMachines",` 部分，其应采用以下格式：

    ```json
    {
      "ProviderNamespace": "Microsoft.Compute",
      "ResourceTypeName": "virtualMachines",
      "Locations": [
        "local"
      ],
      "ApiVersions": [
        "2020-06-01",
        "2019-12-01",
        "2019-07-01",
        "2019-03-01",
        "2018-10-01",
        "2018-06-01",
        "2018-04-01",
        "2017-12-01",
        "2017-03-30",
        "2016-08-30",
        "2016-03-30",
        "2015-11-01",
        "2015-06-15"
      ],
      "ApiProfiles": [
        "2017-03-09-profile",
        "2018-03-01-hybrid"
      ]
    },
    ```

1. 切换到 **sampletemplate2.json** 文件，然后将在上一任务中修改过的 REST API 版本值更改为原始值。确保此版本与上述 **AzureCloudCapabilities.json** 中的其中一个 API 版本匹配，然后保存更改。
1. 切换到 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口运行以下命令，验证最近修改的模板：

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 查看验证的输出并注意是否没有问题。

>**回顾**： 在本练习中，你创建了一个云功能文件，并使用该文件验证了 Azure 资源管理器模板。你还根据验证结果修改了模板。
