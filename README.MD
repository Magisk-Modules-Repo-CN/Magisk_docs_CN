# Magisk文档
(于2018.1.8更新) ([更新日志](changelog.md))

## 目录

- [简介](#introduction)
- [提示与技巧](tips.md)
    - [OTA增量更新提示](tips.md#ota-installation-tips)
- [翻译人员列表](translators.md)

以下内容供开发者阅读

- [流程图](https://cdn.rawgit.com/topjohnwu/Magisk/7d1082b1cb91db90ed0a29d8b092723fc3d69c58/docs/procedures.html)
- [Magisk技术细节](details.md)
    - [Boot Stages](details.md#boot-stages)
    - [Magic Mount Details](details.md#magic-mount-details)
    - [Simple Mount Details](details.md#simple-mount-details)
- [可用工具](tools.md)
    - [magiskboot](tools.md#magiskboot)
    - [magiskinit](tools.md#magiskinit)
    - [magiskpolicy](tools.md#magiskpolicy)
    - [magisk](tools.md#magisk)
    - [su](tools.md#su)
    - [resetprop](tools.md#resetprop)
    - [magiskhide](tools.md#magiskhide)
- [模块](modules.md)
    - [模块与模板](modules.md#magisk-module-format)
    - [提交模块到Repo](https://github.com/topjohnwu/Magisk_Repo_Submissions)
- [提示与技巧](tips.md)
    - [删除文件](tips.md#remove-files)
    - [删除文件夹](tips.md#remove-folders)

    
## 简介
Magisk是一种运行在Android版本大于5.0 Lollipop (API 21).的设备上的开源工具套件。它建立了一个包含大多数基础包的可用于定制Android的环境，例如root， boot scripts， SELinux patches， AVB2.0 / dm-verity / forceencrypt patches 等等……

此外，Magisk 提供 **Systemless Interface** 以在保持真实分区完整性的前提下随时修改system （或 vendor），所有这些都仅仅是通过修补boot映像来完成的。 得益于其自带的systemless与一些篡改手段，Magisk可以隐藏系统修改，并几乎可以绕过所有市面上的系统完整性校验，主要目标之一就是绕过[Google's SafetyNet API](https://developer.android.com/training/safetynet/index.html)。
