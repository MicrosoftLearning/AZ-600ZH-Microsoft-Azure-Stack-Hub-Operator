---
lab:
    title: '实验室：通过使用 Azure Gallery Packager 添加自定义市场项'
    module: '模块 2：提供服务'
---

# 实验室 - 通过使用 Azure Gallery Packager 添加自定义市场项
# 学生实验室手册

## 实验室依赖项

- 无 

## 预计用时

45 分钟

## 实验室场景

你是 Azure Stack Hub 环境的操作员。你需要使用 Azure Gallery Packager 工具创建自定义 Azure Stack 市场项。

## 目标

完成本实验室后，你将能够：

- 使用 Azure Gallery Packager 创建自定义 Azure Stack Hub 市场项

## 实验室环境
 
本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

实验室环境由以下部分组成：

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

在本实验室过程中，你将下载并安装创建自定义 Azure Stack 市场项所需的软件。

### 练习 1：自定义并发布 Azure Stack Hub 市场项

在本练习中，你将使用 Azure Gallery Packager 工具自定义并发布 Azure 市场项。

1. 下载 Azure Gallery Packager 工具和示例包
1. 修改现有的 Azure Gallery Packager 包
1. 将包上传到 Azure Stack Hub 存储帐户 
1. 将包发布到 Azure Stack Hub 市场
1. 验证已发布的 Azure Stack Hub 市场项的可用性

#### 任务 1：下载 Azure Gallery Packager 工具和示例包文件

在此任务中，你将：

- 下载 Azure Gallery Packager 工具和示例包文件

1. 如果需要，请使用以下凭据登录到 **AzS-HOST1**：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码： **Pa55w.rd1234**

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开 Web 浏览器窗口，导航到 [Azure Gallery Packager 工具下载页面](https://aka.ms/azsmarketplaceitem)，然后将 **Microsoft Azure Stack Gallery Packaging Tool 和 Sample 3.0.zip** 存档文件下载到 **Downloads** 文件夹。
1. 下载完成后，将 zip 文件中的 **Packager** 文件夹解压缩到 **C:\\Downloads** 文件夹（如有需要，请创建该文件夹）。


#### 任务 2：修改现有的 Azure Gallery Packager 包

在此任务中，你将：

- 修改现有的 Azure Gallery Packager 包，使其使用 Windows Server 2019 映像（而不是 Windows Server 2016）。

1. 在与 **AzS-HOST1** 的远程桌面会话中，通过文件资源管理器，导航到 **C:\\Downloads\\Packager\\Samples for Packager** 文件夹，然后将 **Sample.SingleVMWindowsSample.1.0.0.azpkg** 包复制到 **C:\\Downloads** 文件夹并将其扩展名重命名为 **zip**。
1. 将 **Sample.SingleVMWindowsSample.1.0.0.zip** 存档的内容解压缩到 **C:\\Downloads\\SamplePackage** 文件夹（需先创建该文件夹）。
1. 在文件资源管理器中，导航到 **C:\\Downloads\\SamplePackage\\DeploymentTemplates** 文件夹，然后在记事本中打开 **createuidefinition.json** 文件。
1. 修改文件的 **imageReference** 部分中的 **sku** 参数，使其引用先前下载到 ASDK 实例的 Windows Server 2019 Datacenter Core 映像：

    ```json
    "imageReference": {
      "publisher": "MicrosoftWindowsServer",
      "offer": "WindowsServer",
      "sku": "2019-Datacenter-Core-smalldisk"
    },
    ```

1. 保存更改并关闭记事本。
1. 在文件资源管理器中，导航到 **C:\\Downloads\\SamplePackage\\strings** 文件夹，然后在记事本中打开 **resources.resjson** 文件。
1. 在键值对中设置以下值，以修改 **resources.resjson** 文件的内容：

    - displayName： **Custom Windows Server 2019 Core VM**
    - publisherDisplayName： **Contoso**
    - summary： **Custom Windows Server 2019 Core VM (small disk)**
    - longSummary： **Custom Contoso Windows Server 2019 Core VM (small disk)**
    - description： **<P>自定义的 Windows Server 2019 Azure Stack Hub VM 示例</p><p>基于 Azure Stack Hub 快速启动模板</p>**
    - documentationLink： **文档**

    >**备注**： 这将生成以下内容：

    ```json
    {
      "displayName": "Custom Windows Server 2019 Core VM",
      "publisherDisplayName": "Contoso",
      "summary": "Custom Windows Server 2019 Core VM (small disk)",
      "longSummary": "Custom Contoso Windows Server 2019 Core VM (small disk)",
      "description": "<p>Sample customized Windows Server 2019 Azure Stack Hub VM</p><p>Based on Azure Stack Hub Quickstart template</p>",
      "documentationLink": "Documentation"
    }
    ```

1. 保存更改并关闭记事本。
1. 在文件资源管理器中，导航到 **C:\\Downloads\\SamplePackage** 文件夹，然后在记事本中打开 **manifest.json** 文件。
1. 在键值对中设置以下值，以修改 **manifest.json** 文件的内容：

    - name: **CustomVMWindowsSample**
    - publisher: **Contoso**
    - version: **1.0.1**
    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - publisherLegalName: **Contoso Inc.**
    - uri of the documentation link: **https://docs.contoso.com**

    >**备注**： 这应生成以下内容（从文件的第 2 行开始）：

    ```json
        "$schema": "https://gallery.azure.com/schemas/2015-10-01/manifest.json#",
        "name": "CustomVMWindowsSample",
        "publisher": "Contoso",
        "version": "1.0.1",
        "displayName": "Custom Windows Server 2019 Core VM",
        "publisherDisplayName": "Contoso",
        "publisherLegalName": "Contoso Inc.",
        "summary": "ms-resource:summary",
        "longSummary": "ms-resource:longSummary",
        "description": "ms-resource:description",
        "longDescription": "ms-resource:description",
	    "uiDefinition": {
		"path": "UIDefinition.json"
	},
        "links": [
            { "displayName": "ms-resource:documentationLink", "uri": "https://docs.contoso.com/" }
        ], 
    ```

1. 保存更改并关闭记事本。


#### 任务 3：生成自定义的 Azure Gallery Packager 包

在此任务中，你将：

- 再生成最近自定义的 Azure Gallery Packager 包。

1. 在与 **AzS-HOST1** 的远程桌面会话中，启动 **“命令提示符”**。

1. 从 **“命令提示符”** 运行以下命令来更改当前的目录：

    ```cmd
    cd C:\Downloads\Packager
    ```

1. 从 **“命令提示符”** 运行以下命令，根据在上一任务中修改的内容生成新的包：

    ```cmd
    AzureStackHubGallery.exe package -m C:\Downloads\SamplePackage\manifest.json -o C:\Downloads\
    ```

1. 验证 **Contoso.CustomVMWindowsSample.1.0.1.azpkg** 包是否已自动保存到 **C:\\Downloads** 文件夹。


#### 任务 4：将包上传到 Azure Stack Hub 存储帐户

在此任务中，你将：

- 将 Azure Stack Hub 市场项包上传到 Azure Stack Hub 存储帐户。

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“+ 创建资源”**。 
1. 在 **“新建”** 边栏选项卡上，单击 **“数据 + 存储”**。
1. 在 **“数据 + 存储”** 边栏选项卡上，单击 **“存储帐户”**。
1. 在 **“创建存储帐户”** 边栏选项卡的 **“基本信息”** 选项卡中，指定以下设置：

    - 订阅： **默认提供商订阅**
    - 资源组：新资源组名称 **marketplace-pkgs-RG**
    - 名称：由 3 - 24 个小写字母或数字组成的唯一名称
    - 位置： **本地**
    - 性能： **标准**
    - 帐户种类： **存储（常规用途 v1）**
    - 复制： **本地冗余存储 (LRS)**

1. 在 **“创建存储帐户”** 边栏选项卡的 **“基本信息”** 选项卡上，单击 **“下一步: 高级 >”**。
1. 在 **“创建存储帐户”** 边栏选项卡的 **“高级”** 选项卡上，保留默认设置，然后单击 **“查看 + 创建”**。
1. 在 **“创建存储帐户”** 边栏选项卡的 **“查看 + 创建”** 选项卡上，单击 **“创建”**。

    >**备注**： 等待存储帐户预配完成。此操作大约需要 1 分钟。

1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口的“中心”菜单中，选择 **“资源组”**。
1. 在 **“资源组”** 边栏选项卡的资源组列表中，单击 **“marketplace-pkgs-RG”** 条目。
1. 在 **“marketplace-pkgs-RG”** 边栏选项卡上，单击表示新创建的存储帐户的条目。
1. 在“存储帐户”边栏选项卡上，单击 **“容器”**。
1. 在“容器”边栏选项卡上，单击 **“+ 容器”**。
1. 在 **“新建容器”** 边栏选项卡的 **“名称”** 文本框中，键入 **gallerypackages**，在 **“公共访问级别”** 下拉列表中，选择 **“Blob(仅对 Blob 进行匿名读取访问)”**，然后单击 **“创建”**。
1. 返回到“容器”边栏选项卡，单击表示新创建的容器的 **gallerypackages** 条目。
1. 在 **“gallerypackages”** 边栏选项卡上，单击 **“上传”**。
1. 在 **“上传 Blob”** 边栏选项卡上，单击 **“选择文件”** 文本框旁的文件夹图标。 
1. 在 **“打开”** 对话框中，导航到 **C:\Downloads** 文件夹，选择 **Contoso.CustomVMWindowsSample.1.0.1.azpkg** 包文件，然后单击 **“打开”**。
1. 返回到 **“上传 Blob”** 边栏选项卡，单击 **“上传”**。


#### 任务 5：将包发布到 Azure Stack Hub 市场

在此任务中，你将：

- 将包发布到 Azure Stack Hub 市场。

1. 在与 **AzS-HOST1** 的远程桌面会话中，以管理员身份启动 PowerShell 7。
1. 在与 **AzS-HOST1** 的远程桌面会话中，从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，安装本实验室所需的 Azure Stack Hub PowerShell 模块： 

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**备注**： 忽略有关已可用命令的任何错误消息。

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，注册 Azure Stack Hub 操作员环境：

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. 从 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 提示符运行以下命令，通过与 Azure Stack Hub 管理员门户的现有浏览器会话，使用 AzureStack\CloudAdmin 凭据登录到 Azure Stack Hub 操作员 PowerShell 环境：

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

    >**备注**： 这将自动打开另一个浏览器选项卡，其中显示了表示身份验证成功的消息。

1. 关闭浏览器选项卡，切换回 **“Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 窗口，并验证是否已成功使用 **CloudAdmin@azurestack.local** 进行身份验证。
1. 从“Administrator: **Administrator: C:\Program Files\PowerShell\7\pwsh.exe”** 运行以下命令，将包发布到 Azure Stack Hub 市场（其中 `<storage_account_name>` 占位符表示在上一任务中分配的存储帐户的名称）：

    ```powershell
    Add-AzsGalleryItem -GalleryItemUri `
    https://<storage_account_name>.blob.local.azurestack.external/gallerypackages/Contoso.CustomVMWindowsSample.1.0.1.azpkg -Verbose
    ```

1. 查看 **Add-AzsGalleryItem** 命令的输出，以验证命令是否成功执行。


#### 任务 6：验证已发布的 Azure Stack Hub 市场项的可用性

在此任务中，你将：

- 验证已发布的 Azure Stack Hub 市场项的可用性。

1. 在与 **AzS-HOST1** 的远程桌面会话中，打开显示 [Azure Stack Hub 用户门户](https://portal.local.azurestack.external)的 Web 浏览器窗口。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击 **“市场”**。 
1. 在 **“市场”** 边栏选项卡中，单击 **“计算”**，然后单击 **“查看更多”**。
1. 在 **“市场”** 边栏选项卡上，确保选项列表中显示**自定义 Windows Server 2019 Core VM**。 

    >**备注**： 为了确保新的 Azure Stack Hub 市场项按预期运行，你还需确保符合其布署的所有先决条件，例如其模板引用的 OS 映像。这超出了本实验室的范围。

>**回顾**： 在本练习中，你使用 Azure Gallery Packager 创建了自定义 Azure Stack Hub 市场项，并使用 **Add-AzsGalleryItem** cmdlet 发布了该市场项。