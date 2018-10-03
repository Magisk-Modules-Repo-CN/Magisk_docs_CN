## 可用工具
Magisk附带了许多安装工具，包括用于保持程序运行的程序、开发人员的实用程序。 本文档包含3个二进制文件，还有许多工具可用作小程序（Applet）。工具之间的关系如下所示：

```
magiskboot                 /* binary */
magiskinit                 /* binary */
magiskpolicy -> magiskinit
supolicy -> magiskinit     /* alias of magiskpolicy */
magisk                     /* binary */
magiskhide -> magisk
resetprop -> magisk
su -> magisk
```

### magiskboot
一个能够【解压缩或重新打包启动映像、解析和修补cpio和dtbs、十六进制转二进制，使用多种算法压缩/解压缩】的工具。它用于安装Magisk 到启动映像中。

`magiskboot` 原生支持（这意味着不必调用外部工具）所有流行的压缩方法，包括 `gzip` （用于压缩内核和ramdisk），lz4（用于压缩像Google Pixel 这样的先进设备中的内核），lz4_legacy（仅使用特殊元数据的传统LZ4块格式）【只用于LG】，lzma（Linux内核本身就支持的LZMA1算法，用于某些自定义内核压缩ramdisk），xz（LZMA2算法，拥有非常高的压缩率，在Magisk中用于高压缩模式和存储二进制文件），以及bzip2（用于桌面Linux启动映像创建bzImage，不过并未应用于Android）。

magiskboot的意图是尽可能保持镜像完整。对于解包，它会提取大块数据（内核，ramdisk，secound，dtb等），并尽可能解压缩它们。重新打包启动镜像时，必须提供原始boot映像，以便它可以使用原始的headers（包括MTK特定headers），只需更改必要的数据（如数据块大小），并使用原始压缩率重新压缩所有数据。同样的概念也适用于CPIO修补：它不提取所有文件，在文件系统中修改，将所有文件存档回cpio，不像通常用于创建Linux一样来initramfs，而是直接在内存中的cpio级别进行修改，并不涉及任何数据的直接修改。

指令帮助信息:

```
Usage: magiskboot <action> [args...]

Supported actions:
 --parse <bootimg>
  Parse <bootimg> only, do not unpack. Return values: 
    0:OK   1:error   2:insufficient boot partition size
    3:chromeos   4:ELF32   5:ELF64

 --unpack <bootimg>
  Unpack <bootimg> to kernel, ramdisk.cpio, (second), (dtb), (extra) into
  the current directory. Return value is the same as --parse

 --repack <origbootimg> [outbootimg]
  Repack kernel, ramdisk.cpio[.ext], second, dtb... from current directory
  to [outbootimg], or new-boot.img if not specified.
  It will compress ramdisk.cpio with the same method used in <origbootimg>,
  or attempt to find ramdisk.cpio.[ext], and repack directly with the
  compressed ramdisk file

 --hexpatch <file> <hexpattern1> <hexpattern2>
  Search <hexpattern1> in <file>, and replace with <hexpattern2>

 --cpio <incpio> [commands...]
  Do cpio commands to <incpio> (modifications are done directly)
  Each command is a single argument, use quotes if necessary
  Supported commands:
    rm [-r] ENTRY
      Remove ENTRY, specify [-r] to remove recursively
    mkdir MODE ENTRY
      Create directory ENTRY in permissions MODE
    ln TARGET ENTRY
      Create a symlink to TARGET with the name ENTRY
    mv SOURCE DEST
      Move SOURCE to DEST
    add MODE ENTRY INFILE
      Add INFILE as ENTRY in permissions MODE; replaces ENTRY if exists
    extract [ENTRY OUT]
      Extract ENTRY to OUT, or extract all entries to current directory
    test
      Test the current cpio's patch status. Return value: 
      0:stock   1:Magisk   2:other (phh, SuperSU, Xposed)
    patch KEEPVERITY KEEPFORCEENCRYPT
      Ramdisk patches. KEEP**** are boolean values
    backup ORIG [SHA1]
      Create ramdisk backups from ORIG
      SHA1 of stock boot image is optional
    restore
      Restore ramdisk from ramdisk backup stored within incpio
    magisk ORIG HIGHCOMP KEEPVERITY KEEPFORCEENCRYPT [SHA1]
      Do Magisk patches and backups all in one step
      Create ramdisk backups from ORIG
      HIGHCOMP, KEEP**** are boolean values
      SHA1 of stock boot image is optional
    sha1
      Print stock boot SHA1 if previously stored

 --dtb-<cmd> <dtb>
  Do dtb related cmds to <dtb> (modifications are done directly)
  Supported commands:
    dump
      Dump all contents from dtb for debugging
    test
      Check if fstab has verity/avb flags. Return value:
      0:no flags    1:flag exists
    patch
      Search for fstab and remove verity/avb

 --compress[=method] <infile> [outfile]
  Compress <infile> with [method] (default: gzip), optionally to [outfile]
  <infile>/[outfile] can be '-' to be STDIN/STDOUT
  Supported methods: gzip xz lzma bzip2 lz4 lz4_legacy 

 --decompress <infile> [outfile]
  Detect method and decompress <infile>, optionally to [outfile]
  <infile>/[outfile] can be '-' to be STDIN/STDOUT
  Supported methods: gzip xz lzma bzip2 lz4 lz4_legacy 

 --sha1 <file>
  Print the SHA1 checksum for <file>

 --cleanup
  Cleanup the current working directory
```

### magiskinit
This tool is created to unify Magisk support for both legacy "normal" devices and new `skip_initramfs` devices. The compiled binary will replace `init` in the ramdisk, so things could be done even before `init` is started.

`magiskinit` is responsible for constructing a proper rootfs on devices which the actual rootfs is placed in the system partition instead of ramdisk in `boot.img`, such as the Pixel familiy and most Treble enabled devices, or I like to call it `skip_initramfs` devices: it will parse kernel cmdline, mount sysfs, parse through uevent files to make the system (or vendor if available) block device node, then copy rootfs files from system. For normal "traditional" devices, it will simply swap `init` back to the original one and continue on to the next stage.

With a proper rootfs, `magiskinit` goes on and does all pre-init operations to setup a Magisk environment. It patches rootfs on the fly, providing fundamental support such as patching `init`, `init.rc`, run preliminary `sepolicy` patches, and extracts `magisk` and `init.magisk.rc` (these two files are embedded into `magiskinit`). Once all is done, it will spawn another process (`magiskinit_daemon`) to asynchronously run a full `sepolicy` patch, then starts monitoring the main Magisk daemon to make sure it is always running (a.k.a invincible mode); at the same time, it will execute the original `init` to hand the boot process back.

### magiskpolicy
(This tool is aliased to `supolicy` for compatibility with SuperSU's sepolicy tool)

This tool is an applet of `magiskinit`: once `magiskinit` had finished its mission in the pre-init stage, it will preserve an entry point for `magiskpolicy`. This tool could be used for advanced developers messing with `sepolicy`, a compiled binary containing SELinux rules. Normally Linux server admins directly modifies the SELinux policy sources (`*.te`) and recompile the `sepolicy` binary, but here we directly patch the binary file since we don't have access to the sources.

All processes spawned from the Magisk daemon, including root shells and all its forks, are running in the context `u:r:su:s0`. Magisk splits the built in patches into 2 parts: preliminary and full

- The preliminary patch should allow all Magisk internal procedures to run properly (can be done manually by the `--magisk` option). It also contains quite a few additional patches so most scripts can run in the daemon before the full patch is done
- The full patch adds the rule `allow su * * *` on top of the preliminary rules. This is done because stock Samsung ROMs do not support permissive; adding this rule makes the domain effectively permissive. Due to the concern of greatly increasing the boot time, the Magisk daemon will not wait for this patch to finish until the boot stage `late_start` triggers. What this means is that **only `late_start` service mode is guaranteed to run in a fully patched environment**. For non-Samsung devices it doesn't matter because `u:r:su:s0` is permissive anyways, but for full compatibility, it is **highly recommended to run boot scripts in `late_start` service mode**.

Command help message:

```
Usage: magiskpolicy [--options...] [policystatements...]

Options:
  --live            directly apply patched policy live
  --magisk          built-in rules for a Magisk selinux environment
  --load FILE       load policies from <infile>
  --save FILE       save policies to <outfile>

If no input file is specified, it will load from current policies
If neither --live nor --save is specified, nothing will happen

One policy statement should be treated as one parameter;
this means a full policy statement should be enclosed in quotes;
multiple policy statements can be provided in a single command

The statements has a format of "<action> [args...]"
Use '*' in args to represent every possible match.
Collections wrapped in curly brackets can also be used as args.

Supported policy statements:

Type 1:
"<action> source-class target-class permission-class permission"
Action: allow, deny, auditallow, auditdeny

Type 2:
"<action> source-class target-class permission-class ioctl range"
Action: allowxperm, auditallowxperm, dontauditxperm

Type 3:
"<action> class"
Action: create, permissive, enforcing

Type 4:
"attradd class attribute"

Type 5:
"typetrans source-class target-class permission-class default-class (optional: object-name)"

Notes:
- typetrans does not support the all match '*' syntax
- permission-class cannot be collections
- source-class and target-class can also be attributes

Example: allow { source1 source2 } { target1 target2 } permission-class *
Will be expanded to:

allow source1 target1 permission-class { all-permissions }
allow source1 target2 permission-class { all-permissions }
allow source2 target1 permission-class { all-permissions }
allow source2 target2 permission-class { all-permissions }
```


### magisk
The magisk binary contains all the magic of Magisk, providing all the features Magisk has to offer. When called with the name `magisk`, it works as an utility tool with many helper functions, and also the entry point for `init` to start Magisk services. These helper functions are extensively used by the [Magisk Module Template](https://github.com/topjohnwu/magisk-module-template) and Magisk Manager.

Command help message:

```
Usage: magisk [applet [arguments]...]
   or: magisk [options]...

Options:
   -c                        print current binary version
   -v                        print running daemon version
   -V                        print running daemon version code
   --list                    list all available applets
   --install [SOURCE] DIR    symlink all applets to DIR. SOURCE is optional
   --createimg IMG SIZE      create ext4 image. SIZE is interpreted in MB
   --imgsize IMG             report ext4 image used/total size
   --resizeimg IMG SIZE      resize ext4 image. SIZE is interpreted in MB
   --mountimg IMG PATH       mount IMG to PATH and prints the loop device
   --umountimg PATH LOOP     unmount PATH and delete LOOP device
   --[init service]          start init service
   --unlock-blocks           set BLKROSET flag to OFF for all block devices
   --restorecon              fix selinux context on Magisk files and folders
   --clone-attr SRC DEST     clone permission, owner, and selinux context

Supported init services:
   daemon, post-fs, post-fs-data, service

Supported applets:
    su, resetprop, magiskhide
```

### su
An applet of `magisk`, the MagiskSU entry point, the good old `su` command.

Command help message:

```
Usage: su [options] [-] [user [argument...]]

Options:
  -c, --command COMMAND         pass COMMAND to the invoked shell
  -h, --help                    display this help message and exit
  -, -l, --login                pretend the shell to be a login shell
  -m, -p,
  --preserve-environment        preserve the entire environment
  -s, --shell SHELL             use SHELL instead of the default /system/bin/sh
  -v, --version                 display version number and exit
  -V                            display version code and exit,
                                this is used almost exclusively by Superuser.apk
  -mm, -M,
  --mount-master                run in the global mount namespace,
                                use if you need to publicly apply mounts
```

Note: even though the `-Z, --context` option is not listed above, it actually still exists for compatibility with apps using SuperSU. However MagiskSU will silently ignore the option since it's no more relevant.

### resetprop
An applet of `magisk`, an advanced system property manipulation utility. Here's some background knowledge: 

System properties are stored in a hybrid trie/binary tree data structure in memory. These properties are allowed to be read by many processes (natively via `libcutils`, in shells via the `getprop` command); however, only the `init` process have direct write access to the memory of property data. `init` provides a `property_service` to accept property update requests and acts as a gatekeeper, doing things such as preventing **read-only** props to be overridden and storing **persist** props to non-volatile storages. In addition, property triggers registered in `*.rc` scripts are also handled here.

`resetprop` is created by pulling out the portion of source code managing properties from AOSP and try to mimic what `init` is doing. With some hackery the result is that we have direct access to the data structure, bypassing the need to go through `property_service` to gain arbitrary control. Here is a small implementation detail: the data structure and the stack-like memory allocation method does not support removing props (they are **designed NOT** to be removed); prop deletion is accomplished by detaching the target node from the tree structure, making it effectively invisible. As we cannot reclaim the memory allocated to store the property, this wastes a few bytes of memory but it shouldn't be a big deal unless you are adding and deleting hundreds of thousands of props over and over again.

Due to the fact that we bypassed `property_service`, there are a few things developer should to be aware of:

- `on property:foo=bar` triggers registered in `*.rc` scripts will not be triggered when props are changed. This could be a good thing or a bad thing, depending on what behavior you expect. The default behavior of `resetprop` matches the original `setprop`, which **WILL** trigger events (implemented by deleting the prop and set the props via `property_service`), but there is a flag (`-n`) to disable it if you need this special behavior.
- persist props are stored both in memory and in `/data/property`. By default, deleting props will **NOT** remove it from persistent storage, meaning the prop will be restored after the next reboot; reading props will **NOT** read from persistent storage, as this is the behavior of normal `getprop`. With the flag `-p` enabled, deleting props will remove the prop **BOTH** in memory and `/data/property`; props will be read from **BOTH** in memory and persistent storage.

Command help message:

```
Usage: resetprop [flags] [options...]

Options:
   -h, --help        show this message
   (no arguments)    print all properties
   NAME              get property
   NAME VALUE        set property entry NAME with VALUE
   --file FILE       load props from FILE
   --delete NAME     delete property

Flags:
   -v      print verbose output to stderr
   -n      set properties without init triggers
           only affects setprop
   -p      access actual persist storage
           only affects getprop and deleteprop
```

### magiskhide
An applet of `magisk`, the CLI to control MagiskHide. Use this tool to communicate with the daemon to change MagiskHide settings.

Command help message: 

```
Usage: magiskhide [--options [arguments...] ]

Options:
  --enable          Start magiskhide
  --disable         Stop magiskhide
  --add PROCESS     Add PROCESS to the hide list
  --rm PROCESS      Remove PROCESS from the hide list
  --ls              Print out the current hide list
```
