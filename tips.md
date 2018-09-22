# 技巧与窍门

## OTA增量更新安装提示
Magisk以systemless方式修改系统，这意味着应用官方的OTA增量更新变得更容易了。假如可能的话，这里我会针对不同的设备给出一些教程，以应用OTA增量更新和保存Magisk的安装状态。

**本教程仅适用于Magisk v17.0+**

**注意:为了应用OTA增量更新，你需要确保你没有以任何方式修改 `/system` （和 `/vendor` 假如可用）。甚至将分区重新挂载为可读写状态(rw)会篡改模块校验！！**

#### 前提
1. 请在开发者选项中禁用 *Automatic system updates*，因此未经你确认时不会安装OTA增量更新。
    <img src="images/disable_auto_ota.png" width="250">
1. 当有可用的OTA增量更新时，请先（Magisk Manager → 卸载 → 还原原厂镜像）. **请不要重启你的设备，否则Magisk会被卸载** 这将会100%还原你的原厂镜像（和 dtbo 假如可用） 以通过OTA增量更新的预先校验**你必须在进行以下步骤前进行本步骤。**  
    <img src="images/restore_img.png" width="250">

#### 拥有A/B双分区的设备

由于此类设备拥有两个独立的分区，因此可以将OTA增量更新安装到非活动的位置中，并且使Magisk Manager安装Magisk到更新的分区中。开箱即用的OTA增量更新安装可以无缝进行，同时Magisk可以在OTA后保存安装状态。
1. 在还原原厂镜像后，可以如同往常那样进行OTA增量更新（设置 → 系统 → 系统升级）。
1. 等待更新完全安装完毕（包含第1步与OTA更新的第2步）， **请不要点击重启按钮！！**安装完毕后，到（Magisk Manager → 安装 → 安装至非活动位置）然后安装Magisk到OTA引擎刚刚更新完毕的位置。 
    <img src="images/ota_done.png" width="250"> <img src="images/install_inactive_slot.png" width="250">
1. 在安装完成后，在Magisk Manager中点击重启按钮。掌控一切的Magisk Manager将强制你的设备切换到更新的系统中，绕过一切可能的post-OTA校验。  
    <img src="images/manager_reboot.png" width="250">
1. 在设备重启后，你的设备应该已经完全更新完毕，最重要的是，Magisk仍已安装到最新的系统之中！

#### 支持FlashFire的设备
（假如你正在使用拥有A/B双分区的设备，我 **强烈** 推荐你使用上述方法进行安装，因为它使用官方的OTA安装机制，并且在任何情况下都可以使用）

由Chainfire开发的[FlashFire](https://play.google.com/store/apps/details?id=eu.chainfire.flash)是一个极好的用于安装OTA增量更新，并与此同时能够保持设备root状态的一款app。但是，有很高的概率不适配于你的设备/系统组合。遗憾的是，该款app的开发者Chainfire已不再维护，因此未来不会再兼容新的设备。

1. 在还原原厂镜像后，下载OTA增量更新（设置 → 系统 → 系统升级）， **请不要点击重启以安装**
1. 打开FlashFire，它应该会检测你的OTA增量更新zip。在弹出的对话框中选择OK以让它完成安装。
1. 请使用以下截图所展示的选项。关键步骤是禁用EverRoot（否则它将会安装SuperSU），然后在OTA update.zip（update.zip应该会在之前的步骤中自动生成）安装完成**之后**增加新任务刷写Magisk zip。  
    <img src="images/flashfire.png" width="250">
1. 请点击很大的**Flash（刷写）** 按钮，几分钟之后设备将自动重启并安装更新，同时附带Magisk。

#### 传统 "无A/B双分区"的设备 - 一般会是这种状况
很遗憾，我们真的没有很好的方法在这种设备上去安装OTA增量更新。以下教程将不会保存Magisk安装状态 - 你需要在升级之后手动重新root你的设备，这需要用到PC。这一般是“最佳做法”了。

1. 为了恰当地安装OTA增量更新，你必须在你的设备上安装官方版本的recovery，如果你安装了第三方recovery，你可以从先前的备份中，在网上，又或者是出厂OEM镜像中找到该文件并恢复。  
如果你决定在不修改recovery分区的前提下安装Magisk， 你有几个选择，无论哪种方式，你都会得到一台由Magisk root的设备且recovery还保持出厂预装状态：
    - 假如设备支持，使用 `fastboot boot <recovery_img>` 启动第三方recovery并安装Magisk。 ‘’‘’‘’‘’‘’‘’‘’‘’‘’
    - 假如你有一份预装boot的转储副本，通过Magisk Manager以修补boot镜像的方式安装Magisk，然后通过download 模式 / fastboot 模式 / Odin手动刷写完成安装。
1. 只要你的设备预装有recovery和boot的镜像恢复，下载该OTA，作为一个可选项，一旦你下载了OTA的update.zip，因为你的设备仍然是root状态,你可以寻找一种方法把这个zip复制出来。就个人而言，我从OTA增量更新包中提取了预装的boot镜像和recovery镜像以供将来备用（去通过Magisk Manager校验或重置Recovery等等。）
1. 应用更新然后重启你的设备。这将会使用你设备上官方原厂的OTA安装机制去更新你的设备。
1. 一旦完成之后，你将得到一个更新的100%完整预装的，未root的设备。你将需要手动将Magisk手动刷写回去。如果你想经常地接收到来自原厂的OTA更新，可以考虑使用步骤1中所述的方法在不修改recovery分区的前提下刷写Magisk。

## 删除文件
如何删除systemless的文件？实际上让文件 **消失** 是很复杂的（可能可以做到，但并不值得去花费功夫）。 **用一个虚拟文件替换它就足够了**！ 创建一个相同文件名的空文件并将其放于模块中相同的路径下，它将用虚拟文件覆盖目标文件。

## 删除文件夹
与上述提到的一样，让文件夹实际上 **消失** 并不值得我们去花费功夫。 **用空白文件夹覆盖它就已经是很好的办法了**！ 对于使用[Magisk Module Template](https://github.com/topjohnwu/magisk-module-template)的模块开发者来说，一个很方便有用的技巧是：将要删除的文件夹添加到 `config.sh` 的 `REPLACE` 列表中。如果你的模块没有提供相应的文件夹，它会创建一个空文件夹然后自动添加 `.replace` 到这个空文件夹中，因此虚拟文件夹会完全自动替换在`/system` 中的对应文件夹。
