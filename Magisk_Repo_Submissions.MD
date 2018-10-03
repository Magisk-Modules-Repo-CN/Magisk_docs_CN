# Magisk Repo提交
这是开发人员将Magisk模块提交给[Magisk-Modules-Repo](https://github.com/Magisk-Modules-Repo)的教程文档翻译。

## 说明
### 译者说明
我个人认为，基本的英语文档阅读能力是每一个开发者必须的，我希望大家在阅读本文档后去阅读[官方文档](https://github.com/topjohnwu/Magisk_Repo_Submissions)，毕竟我的水平有限（高一狗）。

### 原文说明
请阅读[完整的文档](https://github.com/Magisk-Modules-Repo-CN/Magisk_docs_CN)。你应该对Magisk和 `git` 提交都有很好的了解。
在您的个人帐户中创建一个新的Github存储库，并将您的模块文件推送到repo。如果您不想从头开始，请先从fork或clone[magisk-module-template](https://github.com/topjohnwu/magisk-module-template) 开始，然后在那里添加文件
只有master分支对用户可见！如果您使用的是模块模板，则必须手动创建master分支。没有master分支的repo将不被接受！
通过创建issues来提交请求。所有请求都由小型服务器自动处理（代码在此repo保留）。New issues的标题应该以[Submission]为首，你应该为你自己的模块的存储库提供一个GitHub链接（现在没有其他网站支持，抱歉）。
提交issues后协作邀请会发送到您的电子邮件，请接受，以便您有权更新您的repo。

## 删除模块
一旦您接受了GitHub上的协作邀请，您就拥有了管理员权限; 这意味着您可以通过GitHub自行删除它。

## 追加信息
·在您接受邀请后，您应该直接在Magisk-Modules-Repo更新您的项目，而不是在您的个人repo中更新！当您的仓库成功克隆到Magisk-Modules-Repo后，您可以删除私人帐户中的仓库，因为它将不再使用。
·您完成对模块的更新后，都必须递增module.prop中的版本号（Versioncode）。Magisk Manager将版本号与本地安装的模块进行比较，以确定是否需要更新。
