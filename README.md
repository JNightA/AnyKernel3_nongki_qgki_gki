
**------当前是nongki分支------**
## AnyKernel3 - 适用于带内存磁盘镜像修改的内核发布的可刷写 Zip 模板
----------------------------------------------------------------------------------
项目集合了nongki(5.4以下)、qgki(5.4)、gki(5.10开始及以上)的AK3刷机包，并将文档翻译成中文，感谢章鱼坤的AK3压缩包来提取刷机脚本。确认内核版本下载对应的分支。
**自用存档，各种刷机脚本太多，避免到时候要用找不到。**
手动操作:确认是否有ramdisk，有就使用magiskboot unpack解压原镜像，将新的ImageT替换调kernel，repack打包后刷入；没有就省略解压步骤

----------------------------------------------------------------------------------
### 作者：osm0sis @ xda-developers ###

“AnyKernel 是一个 update.zip 模板，无论内存磁盘镜像 (ramdisk) 如何，它都可以将任何内核刷写到任何 ROM 上。” - Koush

AnyKernel2 进一步扩展了该格式，允许内核开发者使用一系列包含的命令方法轻松修改底层内存磁盘镜像，以支持内核功能，同时利用属性和变量来自定义其内核的安装体验。AnyKernel3 则整合了 topjohnwu 的 `magiskboot` 工具的强大功能，默认支持更广泛的格式，并通过像 Magisk 一样修补新的 `Image.*-dtb` 文件来自动检测并保留 Magisk root 权限。

_包含一个基于 Galaxy Nexus (tuna) 的脚本作为参考。所有需要编辑的内容都包含在 `anykernel.sh` 文件中。_

## // 属性 / 变量 ##
```
kernel.string=内核名称 by 你的名字 @ xda-developers
do.devicecheck=1
do.modules=1
do.systemless=1
do.cleanup=1
do.cleanuponabort=0
device.name1=maguro
device.name2=toro
device.name3=toroplus
device.name4=tuna
supported.versions=6.0 - 7.1.2
supported.patchlevels=2019-07 -
supported.vendorpatchlevels=2013-07

BLOCK=/dev/block/platform/omap/omap_hsmmc.0/by-name/boot;
IS_SLOT_DEVICE=0;
RAMDISK_COMPRESSION=auto;
PATCH_VBMETA_FLAG=auto;
```

*   `do.devicecheck=1`：指定此选项后，至少需要提供 `device.name1`。这应与您设备 `build.prop` 文件中的 `ro.product.device`、`ro.build.product`、`ro.product.vendor.device` 或 `ro.vendor.product.device` 值相匹配。可以根据需要支持任意数量的 `device.name#` 属性。可以删除任何未使用的空属性。
*   `do.modules=1`：将 `modules` 目录中的 `.ko` 文件内容推送到根目录 (`/`) 下相同的相对位置，并应用正确的权限。在 A/B 分区设备上，这只能作用于当前活动槽位 (active slot)。
*   `do.systemless=1` (需同时设置 `do.modules=1`)：改为将 `modules` 目录的全部内容推送到设备上，创建一个简单的 "ak3-helper" Magisk/KernelSU 模块。这允许开发者有效地替换系统文件，包括 `.ko` 文件。如果当前内核被更改，则内核助手模块会自动移除自身以防止冲突。
*   `do.cleanup=0`：阻止 zip 包移除其在 `/tmp/anykernel`（默认位置）中的工作目录 - 这对于在 adb shell 中调试补丁是否正常工作可能很有用。
*   `do.cleanuponabort=0`：在安装中止的情况下，阻止 zip 包移除其在 `/tmp/anykernel`（默认位置）中的工作目录。
*   `supported.versions=`：将与当前 ROM 的 `build.prop` 中的 `ro.build.version.release` 进行匹配。可以设置为列表或范围。作为列表（一个或多个条目），例如 `7.1.2` 或 `8.1.0, 9`，它将查找精确匹配项；作为范围，例如 `7.1.2 - 9`，它将检查当前版本是否在指定的限制范围内。空格可选，提供的版本值格式应与 `build.prop` 中的格式相同。
*   `supported.patchlevels=` 和 `supported.vendorpatchlevels=`：将分别与当前系统/供应商 `build.prop` 中的 `ro.build.version.security_patch` 和 `ro.vendor.build.security_patch` 进行匹配。它们可以设置为以 `YYYY-MM` 格式表示的闭区间或开区间日期范围，空格可选，例如 `2019-04 - 2019-06`、`2019-04 -` 或 `- 2019-06`，其中后两个例子分别设置了最小或最大限制。

*   `BLOCK=auto`：代替直接的块设备文件路径，启用对设备引导分区的自动检测，用于通用的、非设备特定的 zip 包。也接受任何分区文件名（来自 by-name 目录），例如 `boot`、`recovery` 或 `vendor_boot`。
*   `IS_SLOT_DEVICE=1`：启用对基于槽位设备的活动引导分区后缀的检测，并将此后缀添加到提供的 `BLOCK=` 路径的末尾。也接受 `auto`，用于通用的、非设备特定的 zip 包。
*   `RAMDISK_COMPRESSION=auto`：允许在解包时自动检测格式并重新打包内存磁盘镜像。将 `auto` 改为 `gz`、`lzo`、`lzma`、`xz`、`bz2`、`lz4` 或 `lz4-l` (用于 lz4 legacy) 将强制以该格式重新打包；使用 `cpio` 或 `none` 将（尝试）强制以未压缩格式重新打包。
*   `PATCH_VBMETA_FLAG=auto`：允许在重新打包时自动使用默认的 AVBv2 vbmeta 标志，并使用 Magisk 配置 Canary 23016+。设置为 `0` 强制保留原始 AVBv2 标志中的任何内容；设置为 `1` 强制修补该标志（仅在少数设备上必要）。
*   `CUSTOMDD="<arguments>"`：可以添加此选项以指定额外的 `dd` 参数，适用于需要将其内核直接“黑客”进大型分区（如 `mmcblk0`）的设备，或强制使用 `dd` 进行刷写。
*   `SLOT_SELECT=active|inactive`：可以添加此选项以指定目标槽位。如果省略，默认值仍为 `active`。
*   `NO_BLOCK_DISPLAY=1`：可以添加此选项以禁用输出检测到的最终使用的分区+槽位路径，适用于选择包含自己自定义输出的 zip 包。
*   `NO_MAGISK_CHECK=1`：可以添加此选项以禁用 Magisk 的检测以及相关的内核/dtb 重新修补，适用于不需要此功能的特殊 zip 包。
*   `NO_VBMETA_PARTITION_PATCH=1`：可以添加此选项以跳过使用 `httools` 处理 vbmeta，因为只要 AVB 不强制要求引导阶段分区的验证，GKI 在 verity/verification 开启的情况下是可引导的。

## // 命令方法 ##
```
ui_print "<text>" [...]                   # 在恢复界面打印文本
abort ["<text>" [...]]                   # 中止安装（可附带错误信息）
contains <string> <substring>            # 检查字符串是否包含子字符串
file_getprop <file> <property>           # 从属性文件中获取指定属性的值

set_perm <owner> <group> <mode> <file> [<file2> ...] # 设置文件权限
set_perm_recursive <owner> <group> <dir_mode> <file_mode> <dir> [<dir2> ...] # 递归设置目录及文件权限

dump_boot                                 # 解包引导镜像（boot image）
split_boot                                # 分离引导镜像各部分
unpack_ramdisk                            # 解包内存磁盘镜像 (ramdisk)

backup_file <file>                        # 备份文件
restore_file <file>                       # 恢复文件
replace_string <file> <if search string> <original string> <replacement string> <scope> # 替换文件中的字符串
replace_section <file> <begin search string> <end search string> <replacement string> # 替换文件中匹配的部分
remove_section <file> <begin search string> <end search string> # 移除文件中匹配的部分
insert_line <file> <if search string> <before|after> <line match string> <inserted line> # 在文件中插入行
replace_line <file> <line replace string> <replacement line> <scope> # 替换文件中的行
remove_line <file> <line match string> <scope> # 移除文件中的行
prepend_file <file> <if search string> <patch file> # 在文件开头插入补丁文件内容
insert_file <file> <if search string> <before|after> <line match string> <patch file> # 在文件指定位置插入补丁文件内容
append_file <file> <if search string> <patch file> # 在文件末尾附加补丁文件内容
replace_file <file> <permissions> <patch file> # 替换整个文件（并设置权限）
patch_fstab <fstab file> <mount match name> <fs match type> block|mount|fstype|options|flags <original string> <replacement string> # 修改 fstab 文件条目
patch_cmdline <cmdline entry name> <replacement string> # 修改内核命令行 (cmdline) 条目
patch_prop <prop file> <prop name> <new prop value> # 修改属性文件 (.prop) 中的属性值
patch_ueventd <ueventd file> <device node> <permissions> <chown> <chgrp> # 修改 ueventd 文件中的设备节点规则

repack_ramdisk                            # 重新打包内存磁盘镜像
flash_boot                                # 刷写引导镜像
flash_generic <partition name>            # 刷写通用分区镜像（如 dtbo, vendor_boot）
write_boot                                # 执行重新打包并刷写引导镜像

reset_ak [keep]                           # 重置 AnyKernel 环境（可选保留 ramdisk/patch）
setup_ak                                  # 设置 AnyKernel 环境
```

*   **"if search string" (如果搜索字符串)**：这是用于判断是否需要添加修改的查找字符串，通常是一些表明该修改已经存在的标识。
*   **"cmdline entry name" (cmdline 条目名称)**：在 `patch_cmdline` 函数中，此参数的作用类似于匹配检查，用于定位要更改/添加的 cmdline 条目名称，后跟要替换它的完整条目。
*   **"prop name" (属性名称)**：在 `patch_prop` 中，此参数也用作属性文件 (`prop file`) 中属性的匹配检查，但它只是属性名称，属性值单独指定。
*   类似地，**"line match string" (行匹配字符串)** 和 **"line replace string" (行替换字符串)** 是定位修改位置的搜索字符串；**"begin search string" (起始搜索字符串)** 和 **"end search string" (结束搜索字符串)** 都是必需的，用于选择 `replace_section` 要替换的脚本块的起始行和结束行；**"mount match name" (挂载点匹配名称)** 和 **"fs match type" (文件系统类型匹配)** 都是必需的，用于将 `patch_fstab` 命令缩小到正确的条目。
*   **"scope" (作用域)**：可以指定为 **"global" (全局)**，强制替换/移除 `replace_string`、`replace_line` 或 `remove_line` 所定位字符串/行的所有实例。如果省略或设置为其他值，则执行默认的首次匹配操作。
*   **"before|after" (之前|之后)**：只需指定 **"before" (之前)** 或 **"after" (之后)** 来表示插入行的位置，相对于 **"line match string" (行匹配字符串)**。
*   **"block|mount|fstype|options|flags" (块设备|挂载点|文件系统类型|选项|标志)**：需要指定您想要检查和修改的 fstab 条目的哪个部分（按顺序列出）。

*   `dump_boot` 和 `write_boot` 是解包/重新打包的默认方法。但为了更精细的控制，或完全省略内存磁盘镜像修改（"OG AK" 模式），可以将其拆分为 `split_boot; unpack_ramdisk` 和 `repack_ramdisk; flash_boot`。`flash_generic` 可用于将镜像刷写到相应的分区（如 `dtbo`, `vendor_dlkm`）。在 `write_boot` 中会自动包含对 `dtbo`, `system_dlkm` 和 `vendor_dlkm` 的刷写，但如果使用 "OG AK" 模式或创建仅刷写分区的简单 zip，也可以单独调用它。

*   **创建多分区 zip**：可以通过从 zip 中移除 `ramdisk` 和 `patch` 文件夹，并改为包含以分区命名（不带槽位后缀）的 "-files" 文件夹来实现，例如 `boot-files` + `recovery-files`，或 `kernel-files` + `ramdisk-files`（在一些 Treble 设备上）。这些文件夹包含 `Image.gz`，以及每个分区的 `ramdisk`、`patch` 等子文件夹。要为下一个分区设置，只需为新的目标分区设置 `BLOCK=`（不带槽位后缀）和 `RAMDISK_COMPRESSION=`，并使用 `reset_ak` 命令。

*   **创建多槽位 zip**：可以使用正常的 zip 布局处理活动槽位（当前槽位），然后通过再次设置 `BLOCK=`（不带槽位后缀）、`SLOT_SELECT=inactive` 和目标槽位的 `RAMDISK_COMPRESSION=`，并使用 `reset_ak keep` 命令来重置以处理非活动槽位 (inactive slot)。`keep` 参数将保留 `patch` 和任何添加的内存磁盘镜像文件用于下一个槽位。

*   `backup_file` 可用于测试以确保内存磁盘镜像修改正确完成，对最终用户透明，或在仅修改内存磁盘镜像的 "mod" zip 中。在后一种情况下，`restore_file` 也可用于创建 "restore" zip 来撤销更改，但应谨慎使用，因为底层的修补文件可能会随着 ROM/内核更新而改变。

*   您也可以使用 `ui_print "<text>"` 在修改过程中将消息写回恢复界面 (recovery)，使用 `abort "<text>"` 中止安装（可附带消息），以及使用 `file_getprop "<file>" "<property>"` 和 `contains "<string>" "<substring>"` 来简化脚本中可能需要的字符串测试逻辑。

## // 包含的二进制工具 ##

AK3 仓库默认包含当前 ARM 架构的 `magiskboot`、`magiskpolicy`、`lptools_static`、`httools_static`、`fec`、`snapshotupdater_static` 和 `busybox` 构建版本，以保持基础包体积小。其他架构（如 x86）和可选二进制工具的构建版本可以从最新的 Magisk zip 包，或我（osm0sis）最新的 AIK-mobile 和 FlashIt 包中获取，地址如下：

https://forum.xda-developers.com/t/tool-android-image-kitchen-unpack-repack-kernel-ramdisk-win-android-linux-mac.2073775/ (Android Image Kitchen 线程)
https://forum.xda-developers.com/t/tools-zips-scripts-osm0sis-odds-and-ends-multiple-devices-platforms.2239421/ (杂项工具和脚本线程)

可以放置在 `/tools` 目录中以启用内置扩展功能的可选支持二进制工具包括：
*   `mkbootfs` - 用于损坏的恢复环境 (recovery)，或通过绑定挂载 (bind mount) 到 `/tmp` 支持脚本/应用程序的启动后刷写（已弃用/谨慎使用）
*   `flash_erase`, `nanddump`, `nandwrite` - MTD 块设备支持，适用于 `dd` 命令不足的设备
*   `dumpimage`, `mkimage` - DENX U-Boot uImage 格式支持
*   `mboot` - Intel OSIP Android 镜像格式支持
*   `unpackelf`, `mkbootimg` - Sony ELF kernel.elf 格式支持，为解锁的引导加载程序重新打包为 AOSP 标准 boot.img
*   `elftool` (需配合 `unpackelf`) - Sony ELF kernel.elf 格式支持，为旧的 Sony 设备重新打包为 ELF
*   `mkmtkhdr` (需配合 `unpackelf`) - 为 Sony 设备支持 MTK 设备引导镜像分区头
*   `futility` + `chromeos` 测试密钥目录 - Google ChromeOS 签名支持
*   `boot_signer-dexed.jar` (已弃用) + `avb` 密钥目录 - Google Android Verified Boot 1.0 (AVBv1) 自定义签名支持
*   `rkcrc` - Rockchip KRNL 内存磁盘镜像支持

可选地将 ARM 构建版本移动到 `tools/arm` 并将 x86 构建版本放在 `tools/x86` 中，将为通用的、非设备特定的 zip 包启用架构检测。

## // 使用说明 ##

1.  **放置内核文件**：将最终的内核构建产物（例如 `Image.gz-dtb` 或 `zImage`）放在 zip 包的根目录。设备所需的任何单独的 `dt`、`dtb`、`recovery_dtbo`、`dtbo`、`system_dlkm` 和/或 `vendor_dlkm` 也应放在这里（如果未包含，将回退到原始文件）。
2.  **放置内存磁盘镜像文件**：将任何必需的内存磁盘镜像文件放在 `/ramdisk` 目录下（对于简单的多分区 `vendor_boot v3` 支持，放在 `/vendor_ramdisk`）。将模块文件（带完整路径，如 `/modules/system/lib/modules`）放在 `/modules` 目录下。
3.  **放置补丁文件**：将任何必需的补丁文件（通常是配合 AK3 文件编辑命令的部分文件）放在 `/patch` 目录下（对于 `vendor_boot v3` 支持，放在 `/vendor_patch`）。
4.  **修改 `anykernel.sh`**：
    *   添加您的内核名称 (`kernel.string`)。
    *   设置引导分区位置 (`BLOCK`)。
    *   为任何添加的内存磁盘镜像文件设置权限（使用 `set_perm` 或 `set_perm_recursive`）。
    *   使用命令方法进行任何需要的内存磁盘镜像修改。
    *   （可选）在根目录放置 `banner` 和/或 `version` 文件，以便在刷写时显示。
5.  **打包 zip 文件**：运行命令 `zip -r9 UPDATE-AnyKernel3.zip * -x .git README.md *placeholder`。
    *   **重要**：最终 zip 包中必须保留 `LICENSE` 文件，以符合二进制分发和 AK3 脚本的许可证要求。

**关于签名**：如果您的 zip 需要在强制验证 zip 签名的恢复环境（如 Cyanogen Recovery）中使用，您还需要按照我在此描述的方法对您的 zip 进行签名：
https://forum.xda-developers.com/t/dev-template-complete-shell-script-flashable-zip-replacement-signing-script.2934449/

**最佳实践建议**：
*   任何无法硬编码到内核源代码中的调整（最佳做法是硬编码），应通过额外的 `init.tweaks.rc` 或 `bootscript.sh` 添加，以最小化必要的内存磁盘镜像更改。在较新的设备上，Magisk 允许将这些文件放在 `/overlay.d` 目录下 - 请参考示例。
*   为了获得最广泛的 AK3 兼容性，修改内存磁盘镜像文件通常比替换整个文件更好。

**调试**：如果在刷写 AK3 zip 包时遇到问题，可以在 zip 文件名后添加后缀 `-debugging`（例如 `MyKernel-AnyKernel3-debugging.zip`）。这将启用 `/tmp` 目录的调试 `.tgz` 文件创建，便于在启动后或在桌面环境中检查。

## // 保持更新 ##

现在您已经为您的设备准备好了一个 zip 包，您可能想知道如何使其与最新的 AnyKernel 提交保持同步。AnyKernel2 和 AnyKernel3 经过精心开发，允许您只需放入最新的 `update-binary` 和 `tools` 目录，对于不精通 git 或脚本的初学者来说，就能“正常工作”。但最佳实践方式如下：

1.  在 GitHub 上 Fork (复制) 我的 AnyKernel3 仓库。
2.  `git clone https://github.com/<你的用户名>/AnyKernel3`
3.  `git remote add upstream https://github.com/osm0sis/AnyKernel3` (添加原始仓库为上游)
4.  `git checkout -b <你的设备名>` (为你的设备创建并切换到一个新分支)
5.  将其设置为您的 `<设备名>` zip 包（即，删除您不使用的任何文件夹，如 `ramdisk` 或 `patch`，删除 `README.md`，添加您的 `anykernel.sh`，如果愿意也可以添加您的 `Image.*-dtb`），然后提交所有这些更改 (`git add .` & `git commit -m "Initial commit for <device>"`)。
6.  `git push --set-upstream origin <你的设备名>` (将您的设备分支推送到您的 GitHub 仓库)
7.  对您支持的每个其他设备重复步骤 4-6：`git checkout master` -> `git checkout -b <另一个设备名>` -> 设置 -> `git push --set-upstream origin <另一个设备名>`。

之后，您应该能够从您的 `master` 分支执行 `git pull upstream master`，并根据需要将新的 AK3 提交合并 (`git merge`) 或挑选 (`git cherry-pick`) 到您的设备分支中。

_**更多支持和用法示例，请访问 AnyKernel3 XDA 论坛线程：** https://forum.xda-developers.com/t/dev-template-anykernel3-easily-mod-rom-ramdisk-pack-image-gz-flashable-zip.2670512/_

**祝您使用愉快！**

---
