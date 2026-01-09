# ImmortalWrt x86-64 平台编译指南

## 前置要求

1. **操作系统**: Debian 11 或 Ubuntu（推荐），需要区分大小写的文件系统
2. **硬件要求**:
   - 基于 AMD64 架构的 CPU
   - 至少 4GB RAM
   - 至少 25GB 可用磁盘空间
   - 需要可访问互联网

3. **依赖工具**: 已安装编译所需的依赖包（参考 README.md）

## 编译步骤

### 1. 更新和安装 Feeds

```bash
# 更新所有 feeds
./scripts/feeds update -a

# 安装所有 feeds 中的软件包
./scripts/feeds install -a
```

### 2. 配置编译选项

运行配置菜单：

```bash
make menuconfig
```

在配置菜单中，需要选择以下选项：

#### 2.1 选择目标平台
- 进入 `Target System` → 选择 `x86`
- 进入 `Subtarget` → 选择 `x86_64`
- 进入 `Target Profile` → 选择 `Generic x86/64`（或根据你的需求选择其他配置）

#### 2.2 选择镜像格式（可选）
- 进入 `Target Images` → 可以选择：
  - `Build GRUB images (Linux x86 or x86_64 bootloader)` - 用于 BIOS/UEFI 启动
  - `Build ISO images` - 用于创建可启动的 ISO 镜像
  - `Build VMDK images` - 用于 VMware
  - `Build VDI images` - 用于 VirtualBox
  - `Build VHDX images` - 用于 Hyper-V

#### 2.3 选择软件包（可选）
- 在 `LuCI` 菜单下可以选择 Web 管理界面
- 在 `Network` 菜单下可以选择网络相关软件包
- 根据需要选择其他软件包

#### 2.4 保存配置
- 按 `S` 保存配置
- 按 `Q` 退出配置菜单

### 3. 开始编译

```bash
# 开始编译（单线程，适合调试）
make

# 或者使用多线程编译（推荐，N 为 CPU 核心数）
make -j$(nproc)

# 在 Windows 上，可以使用：
make -j$env:NUMBER_OF_PROCESSORS
```

编译过程可能需要较长时间（1-3 小时，取决于硬件配置）。

### 4. 查找编译结果

编译完成后，镜像文件位于：

```
bin/targets/x86/64/
```

常见的镜像文件类型：
- `openwrt-x86-64-generic-ext4-combined.img.gz` - 完整镜像（ext4 文件系统）
- `openwrt-x86-64-generic-squashfs-combined.img.gz` - 完整镜像（squashfs 文件系统，只读根分区）
- `openwrt-x86-64-generic-squashfs-combined-efi.img.gz` - UEFI 启动镜像
- `openwrt-x86-64-generic-squashfs-combined-efi.iso` - UEFI 启动 ISO
- `openwrt-x86-64-generic-squashfs-rootfs.img.gz` - 仅根文件系统

## 快速配置命令（非交互式）

如果你想快速配置而不使用菜单，可以使用以下命令：

```bash
# 设置目标为 x86_64
cat >> .config << EOF
CONFIG_TARGET_x86=y
CONFIG_TARGET_x86_64=y
CONFIG_TARGET_x86_64_Generic=y
EOF

# 然后运行 make menuconfig 进行其他配置，或直接编译
```

## 常见问题

### Q: 编译失败怎么办？
A: 
1. 检查是否有足够的磁盘空间
2. 检查网络连接（编译过程需要下载源码）
3. 查看错误日志，通常在 `logs/` 目录下
4. 尝试清理后重新编译：`make clean` 或 `make dirclean`

### Q: 如何只编译特定软件包？
A: 使用 `make package/<package-name>/compile` 命令

### Q: 如何清理编译文件？
A: 
- `make clean` - 清理编译文件
- `make dirclean` - 深度清理（包括工具链）

## 使用编译好的镜像

### 写入物理磁盘
```bash
gunzip openwrt-x86-64-generic-squashfs-combined.img.gz
dd if=openwrt-x86-64-generic-squashfs-combined.img of=/dev/sdX bs=4M
```

### 在虚拟机中使用
- **VirtualBox**: 使用 VDI 格式镜像
- **VMware**: 使用 VMDK 格式镜像
- **QEMU/KVM**: 直接使用 img 文件

### 默认登录信息
- 地址: http://192.168.1.1 或 http://immortalwrt.lan
- 用户名: `root`
- 密码: 无（首次登录需要设置）

## 参考资源

- [ImmortalWrt 官方文档](https://github.com/immortalwrt/immortalwrt)
- [OpenWrt 开发文档](https://openwrt.org/docs/guide-developer/start)

