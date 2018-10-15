# Magisk 模块
## Magisk 模块格式
Magisk 模块存放在 `magisk.img`根目录下的文件夹，其框架如下所述：

```
$MOUNTPOINT（挂载点）
├── .
├── .
├── $MODID                  <--- 模块的ID，应与 module.prop 相匹配
│   ├── auto_mount          <--- 如果该文件存在，自动挂载可用
│   ├── disable             <--- 如果该文件存在，自动挂载不可用
│   ├── module.prop         <--- 该文件用于存放模块的标识和属性
│   ├── post-fs-data.sh     <--- 该脚本会在 post-fs-data 中执行
│   ├── remove              <--- 人如果该文件存在，模块会在下一次重启时被移除
│   ├── service.sh          <--- 该脚本会在 late_start 服务中执行
│   ├── system.prop         <--- 该文件会被视为系统核心脚本(system props)加载
│   ├── system              <--- 如果自动挂载启用， Magisk 会 "Magic Mount" 该文件夹
│   │   ├── .
│   │   ├── .
│   │   └── .
│   ├── vendor              <--- 自动生成，一个链接到 $MODID/system/vendor 的符号链接 
│   ├── .                   <--- 一些其他被允许的文件/文件夹
│   └── .		
├── another_module（其他模块）
│   ├── .
│   └── .
├── .
├── .
```
你不需要使用我提供的 Magisk 模块模板去创建一个新模块。只要你按上述框架放置文件，就能够被 Magisk 识别为模块。

## Magisk 模块模板
 **Magisk 模块模板** 点击**[这里](https://github.com/topjohnwu/magisk-module-template)**获取。

它是一个以zip刷入安装的Magisk模块的模板。它旨在可被简单套用，以便任何人都可以轻松创建自己的模块。模板本身已包含最小安装脚本；大多数功能位于外部的 [util_functions.sh](https://github.com/topjohnwu/Magisk/blob/master/scripts/util_functions.sh) 中，它将会与Magisk一起安装，并且无需使用模板就可以跟随Magisk一同升级。

以下是一些你或许希望了解的文件：

- `config.sh`：用作配置文件的简单脚本。在这里可以配置模块 需要/禁用 的特性。关于如何使用模板的详细说明也写在该文件中。
- `module.prop`：该文件包含您的模块的识别ID和属性，包括名称和版本等等。该文件将用于在真实设备上和 [Magisk Modules Repo](https://github.com/Magisk-Modules-Repo) 中识别Magisk模块。
- `common/*`：启动状态(Boot stage)脚本兼 `system.prop`
- `META-INF/com/google/android/update-binary`：实际安装脚本。修改该脚本以启用高级自定义功能。

以下是一些注意事项：

- 模板依赖于外部Magisk脚本，请按照你模块基于的模板版本在`module.prop`中设定正确的 `minMagisk` 值，或者测试模块所支持的Magisk最低版本。
- **Windows用户请看这里！！**所有文本文件的行尾应以**Unix格式**书写。请使例如Sublime, Atom, Notepad++等高级文本编辑工具去编辑**所有**文档，**绝对不要**用Windows记事本。 
- 在 `module.prop`中, `版本号（version）`可以是任意字符串，因此任何充满创意的版本号命名都被允许（如`ultra-beta-v1.1.1.1`）。但是， `版本内部编号（versionCode）` **必须** 必须为整数型数据。该值用于版本比较。
- 请确保你的模块ID **不包含空格**。
