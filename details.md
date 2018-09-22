## 启动模式
如果你正着手较为复杂的项目，你应当更好地对整个项目进行控制。Magisk可以在不同的启动模式中运行脚本，因此你可以按你所想的去精确微调项目。建议你一同阅读流程图(procedure graph)与本文档。

- post-fs mode
    - **该模式以阻塞方式(BLOCKING)运行。启动会直到所有进程加载完毕后或等待10秒后才继续进行。**
    - 仅在大部分分区挂载后发挥作用。 `/data` 或许在 `vold` 执行前不可用。
    - Magisk将会绑定挂载`/cache/magisk_mount/system` 和 `/cache/magisk_mount/vendor`目录下的文件。
    - 这仅仅是**Simple Mount**, 这意味着它将会覆盖已存在的文件，但不能添加/删除文件。
    - 这一部分通常已被弃用 (具体原因将逐一展开)
- post-fs-data mode
    - **该模式以阻塞方式(BLOCKING)运行。启动会直到所有进程加载完毕后或等待10秒后才继续进行。**
    - 在 `/data` 已经准备好后执行（包括 `/data` 已被加密的情况）。
    - 在 Zygote 和系统服务已启动后执行（这意味着所有进程已经加载完毕）。
    - `/data/adb/magisk.img` 将会被合并,整合并挂载到 `MOUNTPOINT=/sbin/.core/img`
    - Magisk将在`$MOUNTPOINT/.core/post-fs-data.d`目录下运行脚本
    - Magisk将运行以下脚本: `$MOUNTPOINT/$MODID/post-fs-data.sh` （已保存在每个模块的目录中）。
    - Magisk将最后 **Magisk Mount（Magisk挂载）** 模块文件。
- late_start service mode
    - **该模式以非阻塞方式(NON-BLOCKING)运行。此模式下Magisk将会与其他进程并行处理。**
    - 仅在 late_start 类被触发时执行。
    - 守护进程将在运行此模式前等待`sepolicy`的完整校验，因此SELinux将会被确保完整校验。
    - 将较为耗时的脚本放于这时执行。若脚本耗费太长时间执行你在`post-fs-data`中的任务，启动进程将会被卡住。
    - **推荐在该模式下运行所有脚本**，除非你的脚本需要在 Zygote 启动前执行某些任务。
    - Magisk将在`$MOUNTPOINT/.core/service.d`目录下运行脚本
    - Magisk将运行以下脚本: `$MOUNTPOINT/$MODID/service.sh` （已保存在每个模块的目录中）。

## Magic Mount 技术细节
### 术语
- **Item**: 一个文件夹，文件，或符号链接
- **Leaf**: 一个在目录结构树末端的item。 它可以是一个文件或一个符号链接
- **`$MODPATH`**: 一个表示模块文件夹路径的变量
- **Source item**: 一个在`$MODPATH/system`目录下的item，例如， `$MODPATH/system/bin/app_process32`是一个source item
- **Existing item**: 一个在实际文件系统中的item，例如， `/system/bin/app_process32`是一个existing item
- **Target item**: source item的对应项。例如， `$MODPATH/system/bin/app_process32`的target item是 `/system/bin/app_process32`

注意: 一个target item **不** 意味着它是一个existing item。一个target item可能并不存在于真实文件系统

### 策略
- 对于source leaf:如果它的target item也是一个existing item，这个existing item将会被该source leaf所覆盖
- 对于source leaf:如果它的target item不是一个existing item，这个source leaf将会被添加到它的target item的路径中
- 对于任意一个不是target item的existing item，它将会保持原状

以上提到的均为经验规则。基本大意是Magic Mount会合并两个文件夹，将`$MODPATH/system`置于`/system`中，一种更加易于理解的方法是将items认为是从`$MODPATH/system`复制到`/system`中的一个简单快速的拷贝(dirty copied)。

但有一个额外的规则将推翻以上的所有策略:

- 对于包含这个`.replace`文件的source folder，该source folder将被视为一个leaf。即在该target folder的items将被完全丢弃，与此同时，该target folder将被该source folder所覆盖。

目录中被命名为`.replace`的文件将 **不会** 被合并，它将直接覆盖目标目录。一种更加易于理解的方法是将其视为抹除了target folder，然后复制了整个文件夹到target path。

### 注意
- 如果你想替换`/vendor`中的文件，请将其置于`$MODPATH/system/vendor`目录下。Magisk将同时接管两个目录，无论vendor是独立的或内含的，开发者无需理会。
- 有时，完全替换文件夹是不可避免的。例如你想替换原装系统中的`/system/priv-app/SystemUI`。在原装系统中，系统应用通常带有预优化文件。 假如你替换的`SystemUI.apk`是deodexed化的 （这是最通常的情况），你想替换整个`/system/priv-app/SystemUI`以确保该文件夹仅包含修改过的`SystemUI.apk`，且其不带有预优化文件。
- 如果你正在使用 [Magisk模块模板](https://github.com/topjohnwu/magisk-module-template)，你可以在`config.sh`文件中列出一个你想替换的文件夹的清单。安装脚本将会为你在列出的文件夹中创建`.replace`文件。

## Simple Mount 技术细节
（注意: 这一部分通常已被弃用，从开始采用 A/B 分区的设备开始，由于OTA增量更新直接在boot阶段应用，所以不再存在专门用于缓存的分区。取而代之，`/cache` 现在指向了 `/data/cache`，这意味着 `post-fs` 模式不再有 `/cache` 的访问权限）

一些文件要求在启动时较早挂载，目前已知的是一些开机动画和一些libs（大多数用户不会替换它们）。你可以直接将你修改好的文件置于`/cache/magisk_mount`的对应路径下。例如，你想替换`/system/media/bootanimation.zip`，复制你新的开机动画zip到 `/cache/magisk_mount/system/media/bootanimation.zip`，然后Magisk将在下次重启时挂载你的文件。Magisk将 **从target file中克隆所有属性** ，包含selinux环境，许可模式，管理员，组。这意味着你无需担心放置在`/cache/magisk_mount`中的元数据：只要将文件复制到正确的位置，重启设备，一切工作就大功告成了!
