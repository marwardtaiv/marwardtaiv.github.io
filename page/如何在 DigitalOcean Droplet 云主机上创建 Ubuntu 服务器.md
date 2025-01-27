在本文中，你将学习如何通过 DigitalOcean 的管理面板创建一个 Ubuntu 服务器，并将其配置为使用 SSH 密钥。完成服务器设置后，你可以将在其上部署的应用程序和网站进行管理。

本教程是 DigitalOcean 云课程的一部分，它指导用户完成将应用程序安全部署到云的所有步骤。如果你正在寻找有关 DigitalOcean Droplet 云主机的文档，请访问我们的 [产品文档](https://www.digitalocean.com/docs/droplets/how-to/create/)。此外，我们还将持续更新 [DigitalOcean 使用教程](https://space.bilibili.com/3546568420493615)。

## 文章内容

### 准备工作

在开始本教程之前，你需要满足以下条件：

- 对命令行有一定的了解。如果你想了解或复习命令行知识，可以访问我们的 [Linux 命令行入门](https://www.digitalocean.com/community/tutorials/a-linux-command-line-primer)。
- 需要有 SSH 密钥，以便与服务器之间建立安全连接。如需通过命令行设置 SSH 密钥，可参考我们的 [如何在 Ubuntu 上设置 SSH 密钥](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)。
- 需要有有效的信用卡或 PayPal 帐户，以便设置 DigitalOcean Droplet 云主机。请注意，本教程适用于最小的 Droplet 云主机产品，月度订阅费用可在我们的 [定价](https://www.digitalocean.com/pricing/) 页面查看。用户可以随时取消 Droplet 云主机服务。

### 步骤 1 — 创建 DigitalOcean 帐户

要访问 DigitalOcean 控制面板并创建 Droplet 云主机，你需要先注册一个 DigitalOcean 帐户。请导航至 [DigitalOcean 新帐户注册页面](https://www.digitalocean.com/?refcode=db9b35360821)。你可以选择通过电子邮件、Google 或 GitHub 注册。如果遇到问题，可以寻求技术支持。

确认帐户后，请提供信用卡或 PayPal 信息。这些信息用于身份验证和防止垃圾邮件。在选择计划并确认订阅之前，我们不会收费，但可能会看到一笔临时预授权费用，这通常会在一周内取消。

一旦信息被接受，你将进入“注册完成”的窗口，随后可以继续下一步。

### 步骤 2 — 设置你的 Droplet

在完成注册后，你将看到一个窗口，点击“让我们做点什么（Let’s make something）”按钮，之后会出现不同 Droplet 选项按钮，选择底部的“转到控制面板（Go to Control Panel）”链接。

点击后，将带你到控制面板。导航到右上角的“创建（Create）”菜单，然后单击“Droplet”，进入 Droplet 创建页面。

### 步骤 3 — 选择操作系统镜像

在该页面，选择要在 Droplet 上安装的操作系统镜像。DigitalOcean 提供了多种 Linux 发行版，由于本教程使用的是 Ubuntu，请选择 **Ubuntu (LTS) x64** 选项。

### 步骤 4 — 选择计划

在 **选择计划** 部分，你可以选择 Droplet 的 RAM、存储空间和 CPU 核心数。如果你正在按照本教程学习，建议选择每月 5 美元的 **基本** 计划。对于多项目需求，用户可以在后台适时调整计划。

### 步骤 5 — 添加块存储（可选）

若需要额外的文件存储空间，可选择添加块存储。这些卷可在同一区域内从一个 Droplet 移动到另一个 Droplet。

### 步骤 6 — 选择数据中心区域

在 **选择数据中心区域** 部分，选择你希望创建 Droplet 的地区。默认情况下，会选择一个合适的选项，但为了确保最佳性能和最低延迟，建议选择最近的数据中心。

### 步骤 7 — 选择其他选项

在 **选择其他选项** 部分，可以启用多项选项以添加功能。这些功能通常是免费的，包括：

- **IPv6**：为 Droplet 启用 IPv6 访问，支持更大的地址空间。
- **用户数据**：你指定的任意数据，会写入 DigitalOcean 的元数据服务。
- **监控**：允许添加 DigitalOcean 代理以采集扩展指标。

如果不需要这些选项，可以留空。

### 步骤 8 — 设置 SSH 身份验证

在此步骤中，你将使用你创建的 SSH 密钥对为你的 Droplet 设置 SSH 身份验证。选择使用 SSH 密钥进行身份验证，然后点击“新建 SSH 密钥”按钮。

在弹出窗口中，提示你复制公共 SSH 密钥并粘贴至指定空间。若尚未创建 SSH 密钥，请根据弹窗的说明或我们的 [如何在 Ubuntu 上设置 SSH 密钥](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04) 进行操作。

复制你的公钥后粘贴，并为密钥命名，完成后点击“添加新 SSH 密钥”。

### 步骤 9 — 完成并创建 Droplet

最后，你可以选择：

- 创建的 Droplet 数量，推荐仅创建一个作为教程示例。
- 命名你的 Droplet，可以使用自动名称或自定义名称。
- 添加标签以帮助组织 Droplet。
- 将 Droplet 分配到项目中以便管理。
- 选择备份，注意这将增加每月费用。

完成选择后，单击“创建 Droplet”。随后，你将看到 Droplet 创建进度，准备好后将显示其 IP 地址。

通过该 IP 地址，你可以使用终端安全地连接到 Droplet，运行以下命令：

bash
ssh root@your_IP_address


若一切顺利，你将以 root 用户身份登录。建议创建一个权限较低的新用户用于日常操作，避免未授权更改。可参考我们的 [初始服务器设置教程](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu)。

## 结论

在本教程中，你已经成功在 Droplet 上设置了 Ubuntu 服务器，并启用了 SSH 访问。若想了解更多可以执行的操作，请查看我们的 [云课程简介](https://www.digitalocean.com/community/tutorials) 及 [DigitalOcean 教程](https://www.aidroplet.cn/blog/tutorial/)。如需注册线上服务，请参考 [野卡](https://bit.ly/bewildcard)。

如果你希望删除你的 Droplet，可访问我们的产品文档，了解详细步骤。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)