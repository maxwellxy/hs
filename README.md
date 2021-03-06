# Intro

my happy days in HS

# FIQ Debug

[参考文章](http://blog.csdn.net/azloong/article/details/45768633)

使用minicom进入FIQ Debug mode

	ctrl+a+z+f 或ctrl+a+f

退出debug模式

	输入console

# USB Sniffing with tcpdump and wireshark

[参考文章http://blog.csdn.net/xiaojsj111/article/details/14127607](http://blog.csdn.net/xiaojsj111/article/details/14127607)

[参考文章http://omappedia.org/wiki/USB_Sniffing_with_tcpdump](http://omappedia.org/wiki/USB_Sniffing_with_tcpdump)

## 安装必要软件

sudo emerge -v net-analyzer/tcpdump

sudo emerge -v net-analyzer/wireshark

## 操作步骤

- 确认内核配置了usbmonitor

```shell
make menuconfig
	Device Drivers -->
		USB Support -->
			USB Monitor --> Select * not M
```
- mount -t debugfs none_debugs /sys/kernel/debug

- 检查是否存在目录 /sys/kernel/debug/usb/usbmon

```shell
#ls /sys/kernel/debug/usb/usbmon
0s  0u  1s  1t  1u  2s  2t  2u  3s  3t  3u
```

- cat /sys/kernel/debug/usb/devices 确定usb的总线号

tcpdump -D
```shell
1.enp1s0 [Up, Running]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.nflog (Linux netfilter log (NFLOG) interface)
5.nfqueue (Linux netfilter queue (NFQUEUE) interface)
6.dbus-system (D-Bus system bus)
7.dbus-session (D-Bus session bus)
8.usbmon1 (USB bus number 1)
9.usbmon2 (USB bus number 2)
```
## 收集数据(以usbmon1为例)

tcpdump -i usbmon1 -w usblog.pcap &

### 停止采集数据

killall tcpdump

### 使用wireshark 参看数据格式

wireshark usblog.pcap

# fastboot 烧写3288镜像(参考RockChip_Uboot_V3.3.pdf)

默认设备是处于未解锁状态的,所以先要解锁设备才可以使用fastboot烧写

## 查看设备解锁状态

1. 首先让设备进入到fastboot状态,可以在uboot命令行下执行fastboot

2. 在主机端查询解锁状态

	fastboot getvar unlocked

	如果提示未发现设备

	fastboot -i 0x2207 getvar unlocked

## 解锁设备
1. 主机端执行 fastboot oem unlock

2. 5秒内继续执行 fastboot oem unlock_accept

3. 机器会重启进入 recovery 恢复出厂设置

4. 再次进入 fastboot,则fastboot getvar unlocked 应该返回"yes"(设备已解锁)

## fastboot烧写镜像

### 烧写uboot(-i 执行设备idVendor, loader对应3288 uboot)

fastboot -i 0x2207 flash loader RK3288UbootLoader_V2.19.09.bin

### 烧写parameter

fastboot -i 0x2207 flash parameter parameter

### 重启进recovery

fastboot -i 0x2207 oem recovery

### 解锁和锁住设备

fastboot oem unlock 解锁

fastboot oem unlock_accept 确认解锁 (需要在 fastboot oem unlock 命令后,5 秒内输入)

fastboot oem lock 锁住设备

# rkflashkit

### fetch source code

sudo mkdir -p /usr/local/portage/sys-apps/rkflashkit

sudo ebuild /usr/local/portage/sys-apps/rkflashkit/rkflashkit-0.1.4.ebuild manifest

sudo emerge -v sys-apps/rkflashkit

### rkflashkit usage example
sudo rkflashkit part

sudo rkflashkit flash @kernel kernel.img @resource resource.img reboot

# rkflashtool

### fetch source code

git@github.com:linux-rockchip/rkflashtool.git

### compile

make

### usage

rkflashtool b                         reboot device

rkflashtool r partname > file          read flash partition

rkflashtool w partname < file          write flash partition

# depend
[I] dev-libs/libusb 1.0.19-r1

[I] dev-libs/libusb-compat 0.1.5-r2

#命令行启动android应用程序

[参考文章http://blog.csdn.net/wlsfling/article/details/42527083](http://blog.csdn.net/wlsfling/article/details/42527083)

[参考文章http://www.cnblogs.com/greatverve/archive/2012/02/10/android-am.html](http://www.cnblogs.com/greatverve/archive/2012/02/10/android-am.html)

###使用am命令

```shell
# am start -n ｛(package)包名｝/｛包名｝.{活动(activity)名称}
```

程序的入口类可以从每个应用的AndroidManifest.xml的文件中得到,以计算器(calculator)为例,它的
AndroidManifest.xml文件位于packages/apps/Calculator/AndroidManifest.xml

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"

package="com.android.calculator2"

<activity android:name="Calculator"
```

由此计算器(calculator)的启动方法为:
```shell
# am start -n com.android.calculator2/com.android.calculator2.Calculator
```

照相机的manefest在android4.4上如下

packages/apps/Camera2/AndroidManifest.xml

```xml
package="com.android.camera2"

<acttivity
	android:name="com.android.camera.CameraActivity"
```
package名为com.android.camera2

activity名为com.android.camera.CameraActivity

开启照相机
```shell
am start -n com.android.camera2/com.android.camera.CameraActivity
```

```shell
播放本地视频
# am start -a android.intent.action.VIEW -d your_video_file.mp4 -t "video/*"
```

# uboot中宏U_BOOT_CMD定义

```c
#define ll_entry_declare(_type, _name, _list)				\
	_type _u_boot_list_2_##_list##_2_##_name __aligned(4)		\
			__attribute__((unused,				\
			section(".u_boot_list_2_"#_list"_2_"#_name)))

#define U_BOOT_CMD_MKENT_COMPLETE(_name, _maxargs, _rep, _cmd,		\
				_usage, _help, _comp)			\
		{ #_name, _maxargs, _rep, _cmd, _usage,			\
			_CMD_HELP(_help) _CMD_COMPLETE(_comp) }

#define U_BOOT_CMD_MKENT(_name, _maxargs, _rep, _cmd, _usage, _help)	\
	U_BOOT_CMD_MKENT_COMPLETE(_name, _maxargs, _rep, _cmd,		\
					_usage, _help, NULL)

#define U_BOOT_CMD_COMPLETE(_name, _maxargs, _rep, _cmd, _usage, _help, _comp) \
	ll_entry_declare(cmd_tbl_t, _name, cmd) =			\
		U_BOOT_CMD_MKENT_COMPLETE(_name, _maxargs, _rep, _cmd,	\
						_usage, _help, _comp);

#define U_BOOT_CMD(_name, _maxargs, _rep, _cmd, _usage, _help)		\
	U_BOOT_CMD_COMPLETE(_name, _maxargs, _rep, _cmd, _usage, _help, NULL)

/*
 * Usage
 * gcc -E uboot_cmd.c
 */
int main(int argc, char *argv[])
{
	U_BOOT_CMD(mmcinfo, 1, 0, do_mmcinfo, "short help", "long help");
	return 0;
}

#if 0
typedef struct cmd_tbl_s	cmd_tbl_t;
struct cmd_tbl_s {
	char		*name;		/* Command Name			*/
	int		maxargs;	/* maximum number of arguments	*/
	int		repeatable;	/* autorepeat allowed?		*/
					/* Implementation function	*/
	int		(*cmd)(struct cmd_tbl_s *, int, int, char * const []);
	char		*usage;		/* Usage message	(short)	*/
#ifdef	CONFIG_SYS_LONGHELP
	char		*help;		/* Help  message	(long)	*/
#endif
#ifdef CONFIG_AUTO_COMPLETE
	/* do auto completion on the arguments */
	int		(*complete)(int argc, char * const argv[], char last_char, int maxv, char *cmdv[]);
#endif
};

所以宏U_BOOT_CMD(mmcinfo, 1, 0, do_mmcinfo, "short help", "long help");定义了下面这样一个结构体
 cmd_tbl_t _u_boot_list_2_cmd_2_mmcinfo __aligned(4) __attribute__((unused, section(".u_boot_list_2_""cmd""_2_""mmcinfo"))) = { "mmcinfo", 1, 0, do_mmcinfo, "short help", _CMD_HELP("long help") _CMD_COMPLETE(NULL) };;
#endif
```

# NMOST PMOST

![dt0.png](./pngs/dt0.png)
![js1.png](./pngs/js1.png)
![js.png](./pngs/js.png)
![n.png](./pngs/n.png)
![p.png](./pngs/p.png)
![sdg.png](./pngs/sdg.png)

# Misc

## 分包解压缩

[参考文章](http://blog.csdn.net/xiongmc/article/details/17721533)

### 分包压缩文件

	tar -jvcf - Firefly-RK3399_Android6.0_git_20170218.tar.gz | split -b 1G - rk3399.bz2

### 解压分包文件

	cat rk3399.bz2a* | tar -jx
