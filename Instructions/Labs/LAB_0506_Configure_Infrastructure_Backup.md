---
lab:
    title: '实验室：配置 Azure Stack Hub 基础结构备份'
    module: '模块 5：管理基础结构'
---

# 实验室 - 配置 Azure Stack Hub 基础结构备份
# 学生实验室手册

## 实验室依赖项

- 无

## 预计用时

30 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要为基础结构备份做好准备。 

## 目标

完成本实验室后，你将能够：

- 配置 Azure Stack Hub 基础结构备份

## 实验室环境 

本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

实验室环境由以下部分组成：

- 在具有以下接入点的 **AzSHOST-1** 服务器上运行的 ASDK 部署：

  - 管理员门户：https://adminportal.local.azurestack.external
  - 管理员 ARM 终结点：https://adminmanagement.local.azurestack.external
  - 用户门户：https://portal.local.azurestack.external
  - 用户 ARM 终结点：https://management.local.azurestack.external

- 管理用户：

  - ASDK 云操作员用户名： **CloudAdmin@azurestack.local**
  - ASDK 云操作员密码： **Pa55w.rd1234**
  - ASDK 主机管理员用户名：**AzureStackAdmin@azurestack.local**
  - ASDK 主机管理员密码： **Pa55w.rd1234**

在本实验室课程中，你将安装通过 PowerShell 管理 Azure Stack Hub 所需的软件。你还将创建其他用户帐户。

### 练习 1：配置 Azure Stack Hub 基础结构备份

在本练习中，你将为配置 Azure Stack Hub 基础结构备份做好准备：

1. 创建备份用户
1. 创建备份共享
1. 生成加密密钥
1. 配置备份控制器

#### 任务 1：创建备份用户

在此任务中，你将：

- 创建备份用户

1. 如果需要，请使用以下凭据登录到 **AzSHOST-1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，单击 **“开始”**，在 **“开始”** 菜单中，单击 **“Windows 管理工具”**，然后在管理工具列表中，双击 **“Active Directory 管理中心”**。
1. 在 **“Active Directory 管理中心”** 控制台中，单击 **“azurestack(本地)”**。
1. 在“详细信息”窗格中，双击 **“用户”** 容器。
1. 在 **“任务”** 窗格的 **“用户”** 部分中，单击 **“新建” -> “用户”**。
1. 在 **“创建用户”** 窗口中，指定以下设置，然后单击 **“确定”**： 

    - 全名： **AzS-BackupOperator**
    - 用户 UPN 登录名： **AzS-BackupOperator@azurestack.local**
    - 用户 SamAccountName： **azurestack\AzS-BackupOperator**
    - 密码：**Pa55w.rd**
    - 密码选项： **“其他密码选项” -> “密码永不过期”**

#### 任务 2：创建备份共享

在此任务中，你将：

- 创建备份共享。 

    >**备注**： 在非实验室场景中，此共享将位于 Azure Stack Hub 部署的外部。为了简单起见，你将直接在 Azure Stack Hub 主机上创建它。

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动文件资源管理器。 
1. 在文件资源管理器中，创建一个新文件夹 **C:\\Backup**。
1. 在文件资源管理器中，右键单击 **Backup** 文件夹，然后在右键单击菜单中，单击 **“属性”**。
1. 在 **“Backup 属性”** 窗口中，单击 **“共享”** 选项卡，然后单击 **“高级共享”**。
1. 在 **“高级共享”** 对话框中，单击 **“共享此文件夹”**，然后单击 **“权限”**。
1. 在 **“Backup 的权限”** 窗口中，确保选中 **“所有人”** 条目，然后单击 **“删除”**。
1. 单击 **“添加”**，在 **“选择用户、计算机、服务帐户或组”** 对话框中，键入 **AzS-BackupOperator**，然后单击 **“确定”**。
1. 确保已选中 **“AzS-BackupOperator”** 条目，然后单击 **“允许”** 列中的 **“完全控制”** 复选框。
1. 单击 **“添加”**，在 **“选择用户、计算机、服务帐户或组”** 对话框中，单击 **“位置”**。
1. 在 **“位置”** 对话框中，单击代表本地计算机 (**AzS-HOST1**) 的条目，然后单击 **“确定”**。
1. 在 **“输入要选择的对象名称”** 文本框中，键入 **“管理员”**，然后单击 **“确定”**。
1. 确保选中 **“管理员”** 条目，单击 **“允许”** 列中的 **“完全控制”** 复选框，然后单击 **“确定”**。
1. 返回 **“高级共享”** 对话框，单击 **“确定”**。
1. 返回 **“Backup 属性”** 窗口，单击 **“安全性”** 选项卡，然后单击 **“编辑”**。
1. 单击 **“添加”**，在 **“选择用户、计算机、服务帐户或组”** 对话框中，键入 **AzS-BackupOperator**，然后单击 **“确定”**。
1. 在 **“Backup 的权限”** 对话框的 **“组或用户名称”** 窗格中的条目列表中，单击 **“AzS-BackupOperator”**，在 **“AzS-BackupOperator 的权限”** 窗格中，单击 **“允许”** 列中的 **“完全控制”**，然后单击 **“确定”**。 
1. 返回 **“Backup 属性”** 窗口，单击 **“关闭”**。

#### 任务 3：生成加密密钥

在此任务中，你将：

- 生成加密密钥。 

    >**备注**： 所有基础结构备份都必须经过加密，因此要配置基础结构备份，必须提供与加密密钥对相对应的证书。你将使用 Windows PowerShell 生成密钥。 

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell 7。
1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，生成加密密钥对和相对应的证书：
    
    ```powershell
    $cert = New-SelfSignedCertificate `
               -DnsName "azsh.contoso.com" `
               -CertStoreLocation 'cert:\LocalMachine\My'

    New-Item -Path 'C:\' -Name 'CertStore' -ItemType 'Directory'

    Export-Certificate `
        -Cert $cert `
        -FilePath C:\CertStore\AzSHIBPK.cer 
    ```

    >**备注**： Azure Stack Hub 支持自签名证书，以用于基础结构备份。请记住，在恢复过程中需要提供私钥，因此在生产环境中，请确保将其存储在安全的位置。


#### 任务 4：配置备份控制器

在此任务中，你将：

- 配置备份控制器

1. 在与 AzS-HOST1 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在 Azure Stack Hub 管理员门户中，单击 **“所有服务”**。
1. 在 **“所有服务”** 边栏选项卡上，选择 **“管理”**，然后选择 **“基础结构备份”**。 
1. 在 **“基础结构备份”** 边栏选项卡上，单击 **“配置”**。
1. 在 **“备份控制器设置”** 边栏选项卡上，指定以下设置，然后单击 **“确定”**：

    - 备份存储位置： **\\AzS-HOST1.azurestack.local\Backup**
    - 用户名： **AzS-BackupOperator@azurestack.local**
    - 密码： **Pa55w.rd**
    - 确认密码： **Pa55w.rd**
    - 备份频率(小时)： **12**
    - 保留期(天)： **7**
    - 证书 .cer 文件： **C:\CertStore\AzSHIBPK.cer**

1. 若要验证是否已启用基础结构备份，请刷新显示 Azure Stack Hub 管理员门户的 Web 浏览器页面，然后导航回 **“基础结构备份”** 边栏选项卡。
1. 在 **“基础结构备份”** 边栏选项卡上，查看备份设置。
1. （可选）单击 **“立即备份”** 以启动按需备份。

    >**备注**： 备份过程无法暂停或停止，并且需要超过 15 分钟的时间才能完成。

>**回顾**： 在本练习中，你创建了一个将用于托管 Azure Stack Hub 基础结构备份的共享，以及一个 Active Directory 用户帐户，Azure Stack Hub 基础结构备份将使用该帐户连接到该共享、生成的加密密钥和已配置的备份控制器。 