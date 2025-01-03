## 嘉为蓝鲸smartctl插件使用说明

### 插件功能
导出存储设备（如硬盘驱动器和固态硬盘）SMART 信息的 Prometheus 指标。

### 版本支持

操作系统支持: linux, windows

是否支持arm: 支持

**组件支持版本：**

smartctl版本支持: smartmontools >= 7.0

**是否支持远程采集:**

否

### 参数说明

| **参数名**                 | **含义**                                               | **是否必填** | **使用举例**                    |
|-------------------------|------------------------------------------------------|----------|-----------------------------|
| smartctlPath            | `smartctl` 二进制文件的绝对路径(环境变量), 在页面填写时不要带双引号            | 是        | /usr/sbin/smartct           |
| smartctlDevices         | 指定要监控的设备路径和类型(环境变量), 在页面填写时注意要带双引号                   | 否        | /dev/sda;scsi,/dev/sdb;scsi |
| smartctlDeviceExclude   | 排除自动扫描设备的正则表达式(环境变量)（与 `device-include` 互斥）, 默认为""   | 否        | sda                         |
| smartctlDeviceInclude   | 包含在自动扫描设备中的正则表达式(环境变量)（与 `device-exclude` 互斥）, 默认为"" | 否        | sd.*                        |
| smartctlScanDeviceTypes | 在自动扫描时使用的设备类型(用`,`分隔)(环境变量)                          | 否        | scsi,nvme                   |
| --smartctl.scan         | 启用或禁用自动扫描设备(开关参数)（无设备指定时默认启用）                        | 否        |                             |
| --smartctl.interval     | `smartctl` 轮询的时间间隔（如 `60s` 表示每 60 秒轮询一次）             | 否        | 60s                         |
| --smartctl.rescan       | 扫描新设备或消失设备的时间间隔（如 `10m` 表示每 10 分钟一次扫描）               | 否        | 10m                         |
| --log.level             | 日志级别                                                 | 否        | info                        |
| --web.listen-address    | exporter监听id及端口地址                                    | 否        | 127.0.0.1:9601              |

**注意**: 开启自动扫描时, smartctlDeviceExclude、smartctlDeviceInclude、smartctlScanDeviceTypes参数才会生效。  

### 使用指引

#### 安装smartctl

此监控探针依赖 `smartmontools` 包，需要安装后才能使用，安装方法如下：

```shell
# CentOS
yum install smartmontools

# Ubuntu
apt install smartmontools  
```

#### 参数说明

##### smartctlDevices
一般情况下可以不填写指定设备信息参数，此时会自动开启自动扫描功能。
若想手动指定设备，可以填写smartctlDevices参数，但是自动扫描功能会被关闭。
如果需要指定设备，smartctlDevices参数格式为`/dev/sda;scsi`，其中`/dev/sda`为设备路径，`scsi`为设备类型，设备类型可选值如下：
=======> VALID ARGUMENTS ARE: ata, scsi[+TYPE], nvme[,NSID], sat[,auto][,N][+TYPE], usbcypress[,X], usbjmicron[,p][,x][,N], usbprolific, usbsunplus, sntjmicron[,NSID], intelliprop,N[+TYPE], jmb39x,N[,sLBA][,force][+TYPE], marvell, areca,N/E, 3ware,N, hpt,L/M/N, megaraid,N, aacraid,H,L,ID, cciss,N, auto, test <=======   

如果想填写多个设备，用`,`分隔，例如：`/dev/sda;scsi,/dev/sdb;scsi`

##### --smartctl.scan(自动扫描)
如果不指定smartctlDevices, 则会开启自动扫描功能, 此时可以设置smartctlScanDeviceTypes、smartctlDeviceInclude、smartctlDeviceExclude

### FAQ
大部分问题可以查询[FAQ](https://www.smartmontools.org/wiki/FAQ#DosmartctlandsmartdrunonavirtualmachineguestOS)      

#### SMART状态
虚拟机等设备例如`VMware ESXi`可能不支持SMART  
smartctl_device_smart_status为0时需要检查,  smartctl -i /dev/sda 信息是否正常, 例如：   
```shell
smartctl 7.1 2019-12-30 r5022 [x86_64-linux-5.4.0-202-generic] (local build)
Copyright (C) 2002-19, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor:               VMware
Product:              Virtual disk
Revision:             2.0
Compliance:           SPC-4
User Capacity:        53,687,091,200 bytes [53.6 GB]
Logical block size:   512 bytes
LU is thin provisioned, LBPRZ=1
Device type:          disk
Local Time is:        Tue Dec 31 16:15:09 2024 CST
SMART support is:     Unavailable - device lacks SMART capability.
```



### 指标简介
| **指标ID**                                  | **指标中文名**    | **维度ID**                                                                                                                                                                                                          | **维度含义**                                                                                                | **单位**  | **指标类型** |
|-------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|---------|----------|
| smartctl_devices                          | 发现的设备数量      | -                                                                                                                                                                                                                 | -                                                                                                       | -       | gauge    |
| smartctl_version                          | smartctl版本   | build_info, json_format_version, smartctl_version, svn_revision                                                                                                                                                   | 构建版本信息, JSON格式的版本, 当前smartctl工具版本号, SVN修订版号                                                             | -       | gauge    |
| smartctl_device                           | 设备信息         | ata_additional_product_id, ata_version, device, firmware_version, form_factor, interface, model_family, model_name, protocol, sata_version, scsi_product, scsi_revision, scsi_vendor, scsi_version, serial_number | ATA附加产品ID, ATA版本, 设备标识符, 固件版本, 形态规格, 接口, 型号家族, 型号名称, 协议, SATA版本, SCSI产品, SCSI 修订版, SCSI供应商, SCSI版本, 序列号 | -       | gauge    |
| smartctl_device_attribute                 | 设备属性         | attribute_flags_long, attribute_flags_short, attribute_id, attribute_name, attribute_value_type, device                                                                                                           | 长格式属性标志, 短格式属性标志, 属性ID, 属性名称, 属性值类型, 设备标识符                                                              | -       | gauge    |
| smartctl_device_rotation_rate             | 设备旋转速率       | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | rpm     | gauge    |
| smartctl_device_interface_speed           | 设备接口速度       | speed_type, device                                                                                                                                                                                                | 速度类型, 设备标识符                                                                                             | -       | gauge    |
| smartctl_device_available_spare           | 设备可用备用容量百分比  | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | percent | counter  |
| smartctl_device_available_spare_threshold | 设备可用备用容量阈值   | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | percent | counter  |
| smartctl_device_block_size                | 设备块大小        | blocks_type, device                                                                                                                                                                                               | 块类型, 设备标识符                                                                                              | bytes   | gauge    |
| smartctl_device_capacity_blocks           | 设备块容量        | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | gauge    |
| smartctl_device_capacity_bytes            | 设备字节容量       | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | bytes   | gauge    |
| smartctl_device_bytes_read                | 设备已读取字节数     | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | bytes   | counter  |
| smartctl_device_bytes_written             | 设备已写入字节数     | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | bytes   | counter  |
| smartctl_device_power_cycle_count         | 设备通电循环次数     | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | counter  |
| smartctl_device_power_on_seconds          | 设备已通电时间      | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | s       | counter  |
| smartctl_device_critical_warning          | 设备严重警告       | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | counter  |
| smartctl_device_media_errors              | 设备介质错误数      | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | counter  |
| smartctl_device_num_err_log_entries       | 错误日志条目数      | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | counter  |
| smartctl_device_smart_status              | SMART状态      | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | gauge    |
| smartctl_device_smartctl_exit_status      | smartctl退出状态 | device                                                                                                                                                                                                            | 设备标识符                                                                                                   | -       | gauge    |
| smartctl_device_temperature               | 设备温度         | device, temperature_type                                                                                                                                                                                          | 设备标识符, 温度类型                                                                                             | celsius | gauge    |

### 版本日志

#### weops_smartctl_exporter 1.13.0

- weops调整


添加“小嘉”微信即可获取smartctl监控指标最佳实践礼包，其他更多问题欢迎咨询

<img src="https://wedoc.canway.net/imgs/img/小嘉.jpg" width="50%" height="50%">


