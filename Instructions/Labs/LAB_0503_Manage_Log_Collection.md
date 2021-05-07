---
lab:
    title: '实验室：在 Azure Stack Hub 中管理日志收集'
    module: '模块 5：管理基础结构'
---

# 实验室 - 在 Azure Stack Hub 中管理日志收集
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要确定收集日志的方法。

## 目标

完成本实验室后，你将能够：

- 执行 Azure Stack Hub 日志收集

## 实验室环境 

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


### 练习 1：探索 Azure Stack Hub 诊断日志收集功能

在本练习中，你将探索用于管理诊断日志收集的各个选项。

1. 启用主动诊断日志收集
1. 按需发送诊断日志
1. 将诊断日志复制到本地文件共享

#### 任务 1：启用主动诊断日志收集

在此任务中，你将：

- 启用主动日志收集。

>**备注**： 打开支持案例前，主动日志收集会自动收集诊断日志并将其从 Azure Stack Hub 发送到 Microsoft。仅在触发系统运行状况警报时收集这些日志，并且在支持案例中仅 Microsoft 支持部门可访问这些日志。

1. 如果需要，请使用以下凭据登录到 **AzSHOST-1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击主菜单中的 **“帮助 + 支持”**。
1. 在 **“概述”** 边栏选项卡中，单击 **“日志收集”**。
1. 在 **“概述 \| 日志收集”** 边栏选项卡中，单击 **“启用主动日志收集”**。

    >**备注**： 如果 **“启用主动日志收集”** 选项不可用，请直接进行下一步。 

1. 在 **“设置”** 边栏选项卡上，指定以下设置并单击 **“保存”**。

    - 主动日志收集： **启用**
    - 日志位置： **Azure (推荐)**

1. 返回 **“概述 \| 日志收集”** 边栏选项卡，单击 **“设置”** 以验证是否根据指定的设置配置了主动日志收集。


#### 任务 2：按需发送诊断日志

在此任务中，你将：

- 使用 Azure Stack Hub 管理门户按需发送诊断日志。
- 使用 Azure Stack Hub PowerShell 按需发送诊断日志。 

>**备注**： 此功能称为 *“立即发送日志”*

>**备注**： 首先，你将使用 Azure Stack Hub 管理门户按需发送诊断日志。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口的主菜单中，单击 **“帮助 + 支持”**。
1. 在 **“概述”** 边栏选项卡中，单击 **“日志收集”**。
1. 在 **“概述 \| 日志收集”** 边栏选项卡中，单击 **“立即发送日志”**。
1. 在 **“立即发送日志”** 边栏选项卡上，指定以下设置并单击 **“收集 + 上传”**。

    - 开始时间：当前日期和时间 - 3 小时
    - 结束时间：当前日期和时间

    >**备注**： 不要等待上传完成，而是继续执行下一步。日志收集将失败。这是意料之中的，因为无法从实验室环境直接访问目标 Azure 订阅。

    >**备注**： 现在，你将使用 Azure Stack Hub PowerShell 配置等效功能。这需要连接到特权终结点。

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell ISE。
1. 从 **“Administrator: Windows PowerShell ISE”** 控制台中运行以下命令，确定运行特权终结点的基础结构 VM 的 IP 地址：

    ```powershell
    $ipAddress = (Resolve-DnsName -Name AzS-ERCS01).IPAddress
    ```

1. 从 **“Administrator: Windows PowerShell ISE”** 窗口运行以下命令，将运行特权终结点的基础结构 VM 的 IP 地址添加到 WinRM 受信任主机列表中（除非已允许所有主机）：

    ```powershell
    $trustedHosts = (Get-Item -Path WSMan:\localhost\Client\TrustedHosts).Value
    If ($trustedHosts -ne '*') {
	If ($trustedHosts -ne '') {
		$trustedHosts += ",ipAddress"
	} else {
	$trustedHosts = "$ipAddress"
	}
    }
    Set-Item WSMan:\localhost\Client\TrustedHosts -Value $TrustedHosts -Force
    ```

1. 从 **“Administrator: Windows PowerShell ISE”** 窗口运行以下命令，将 Azure Stack Hub 管理员凭据存储在变量中：

    ```powershell
    $adminUserName = 'CloudAdmin@azurestack.local'
    $adminPassword = 'Pa55w.rd1234' | ConvertTo-SecureString -Force -AsPlainText
    $adminCredentials = New-Object PSCredential($adminUserName,$adminPassword)
    ```

1. 从 **“Administrator: Windows PowerShell ISE”** 窗口运行以下命令，建立与特权终结点的 PowerShell 远程处理会话：

    ```powershell
    Enter-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    ```

    >**备注**： 验证是否已成功建立 PowerShell 远程处理会话。“Windows PowerShell ISE”窗口中的“控制台”窗格应显示用方括号括起来的提示，该提示以运行特权终结点的基础结构 VM 的 IP 地址开头。

1. 从 **“Administrator: Windows PowerShell ISE”** 窗口的“控制台”窗格中的 PowerShell 远程处理会话运行以下命令，按需发送 Azure Stack Hub 存储诊断日志：

    ```powershell
    Send-AzureStackDiagnosticLog -FilterByRole Storage
    ```

    >**备注**： 不要等待上传完成，而是继续执行下一步。日志收集将失败。这是意料之中的，因为无法从实验室环境直接访问目标 Azure 订阅。

    >**备注**： 你可以选择按角色筛选，也可以指定应收集日志的时段。有关详细信息，请参阅[诊断日志收集](https://docs.microsoft.com/zh-cn/azure/azure-stack/azure-stack-diagnostics)。


#### 任务 3：将诊断日志复制到本地文件共享

在此任务中，你将：

- 使用 Azure Stack Hub 管理门户将诊断日志复制到本地文件共享。
- 使用 Azure Stack Hub PowerShell 将诊断日志复制到本地文件共享。 

>**备注**： 首先，你将创建一个文件共享来存储日志。

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动文件资源管理器。 
1. 在文件资源管理器中，创建一个新文件夹 **C:\\AzSHLogs**。
1. 在文件资源管理器中，右键单击 **AzSHLogs** 文件夹，然后在右键单击菜单中，单击 **“属性”**。
1. 在 **“AzSHLogs 属性”** 窗口中，单击 **“共享”** 选项卡，然后单击 **“高级共享”**。
1. 在 **“高级共享”** 对话框中，单击 **“共享此文件夹”**，然后单击 **“权限”**。
1. 在 **“AzSHLogs 的权限”** 窗口中，确保选中 **“所有人”** 条目，然后单击 **“删除”**。
1. 单击 **“添加”**，在**“选择用户、计算机、服务帐户或组”** 对话框中，键入 **CloudAdmins**，然后单击**“确定”**。
1. 确保已选中 **“CloudAdmins”** 条目，然后单击 **“允许”** 列中的 **“完全控制”** 复选框。
1. 单击 **“添加”**，在 **“选择用户、计算机、服务帐户或组”** 对话框中，单击 **“位置”**。
1. 在 **“位置”** 对话框中，单击代表本地计算机 (**AzS-HOST1**) 的条目，然后单击 **“确定”**。
1. 在 **“输入要选择的对象名称”** 文本框中，键入**“管理员”**，然后单击 **“确定”**。
1. 确保选中 **“管理员”** 条目，单击 **“允许”** 列中的 **“完全控制”** 复选框，然后单击 **“确定”**。
1. 返回 **“高级共享”** 对话框，单击 **“确定”**。
1. 返回 **“AzSHLogs 属性”** 窗口，单击 **“安全性”** 选项卡，然后单击 **“编辑”**。
1. 单击 **“添加”**，在 **“选择用户、计算机、服务帐户或组”** 对话框中，键入 **CloudAdmins**，然后单击 **“确定”**。
1. 在 **“AzSHLogs 的权限”** 对话框的**“组或用户名称”** 窗格中的条目列表中，单击 **“CloudAdmins”**，在 **“CloudAdmins 的权限”** 窗格中，单击 **“允许”** 列中的 **“完全控制”**，然后单击 **“确定”**。
1. 返回 **“AzSHLogs 属性”** 窗口，单击 **“关闭”**。

    >**备注**： 接下来，你可以从 Azure Stack Hub 管理门户或通过特权终结点将诊断日志收集配置为文件共享。

1. 在与 **AzS-HOST1** 的远程桌面会话中，在显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口的主菜单中，单击 **“帮助 + 支持”**。
1. 在 **“概述”** 边栏选项卡中，单击 **“日志收集”**。
1. 在 **“概述 \| 日志收集”** 边栏选项卡中，单击 **“设置”**。
1. 在 **“设置”** 边栏选项卡上，将 **“日志位置”** 选项从 **“Azure(推荐)”** 更改为 **“本地文件共享”**，并指定以下设置：

    - SMB 文件共享路径： **\\\\AzS-HOST1.azurestack.local\\AzSHLogs**
    - 用户名： **AZURESTACK\\AzureStackAdmin**
    - 密码： **Pa55w.rd1234**

1. 在 **“设置”** 边栏选项卡中，单击 **“保存”**。
1. 返回 **“概述 \| 日志收集”** 边栏选项卡中，单击**“立即发送日志”**。
1. 在 **“立即发送日志”** 边栏选项卡上，指定以下设置并单击 **“收集 + 上传”**。

    - 开始时间：当前日期和时间 - 3 小时
    - 结束时间：当前日期和时间

    >**备注**： 若要查看日志收集和上传的进度，请在 **“概述 \| 日志收集”** 边栏选项卡中，单击 **“刷新”**。

    >**备注**： 不要等待上传完成，而是继续执行下一步。日志收集应该会成功，但是可能需要大约 15 分钟才能完成。 

    >**备注**： 现在，你将使用 Azure Stack Hub PowerShell 配置等效功能。这需要连接到特权终结点。

1. 在与 **AzS-HOST1** 的远程桌面会话中，切换回 **“Administrator: Windows PowerShell”** 控制台，从该控制台连接到上一个任务中的特权终结点。
1. 从 **“Administrator: Windows PowerShell”** 窗口的“控制台”窗格中的 PowerShell 远程处理会话运行以下命令，将 Azure Stack Hub 存储诊断日志复制到本地文件共享：

    ```powershell
    Get-AzureStackLog -OutputSharePath '\\AzS-HOST1.azurestack.local\AzSHLogs' -OutputShareCredential $using:adminCredentials -FilterByRole Storage
    ```

1. 等待该 cmdlet 完成，然后在文件资源管理器中查看 **C:\\AzSHLogs** 文件夹的内容。

    >**备注**： 该文件夹应包含与你启动的每个单独副本相对应的文件夹。这些文件夹的名称应采用 **AzureStackLogs-*YYYYMMDDHHMMSS*-AZS-ERCS01** 格式，其中 **YYYYMMDDHHMMSS** 表示副本的时间戳。

1. 在 **“Administrator: Windows PowerShell”** 窗口的 PowerShell 远程处理会话提示符下，运行以下命令以关闭该会话：

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzS-HOST1.azurestack.local\AzSHLogs' -Credential $using:adminCredentials
    ```

>**回顾**： 在本练习中，你建立了与特权终结点的 PowerShell 远程处理会话，并使用了该终结点提供的 PowerShell cmdlet 来收集诊断日志。