# 小易智慧公司企业级网络集成项目

> **一个融合了高级路由、交换、安全与无线技术的综合网络解决方案**

## 项目简介

本项目为模拟科技企业"小易智慧公司"设计并完整实施了一个高标准的企业网络。网络覆盖总部8大职能部门、数据中心及无线休闲区域，核心目标是构建一个**高性能、高可用、高安全**的现代化园区网。

**仿真平台:** Cisco Packet Tracer 8.2.0

## 🔥 核心技术与特性

本项目并非简单的拓扑连接，而是集成了以下关键企业级技术：

- **架构:** 三层网络架构（核心-汇聚-接入）
- **冗余:** HSRP (网关冗余)
- **路由:** OSPF (动态路由协议)
- **安全:** VLAN间ACL、NAT、端口安全
- **无线:** 集中式WLC管理 (CAPWAP)
- **运维:** DHCP (自动地址分配)
- **设计:** 专业的IP地址与VLAN规划

## 📊 网络设计摘要

### VLAN与IP地址规划

本项目采用规范的IP地址规划，IP网段与VLAN ID直接关联，便于管理和故障排除。

| VLAN ID | 部门/区域     | IP网段           | 网关         | HSRP组 |
|---------|---------------|------------------|--------------|--------|
| 10      | 财务部        | 192.168.11.0/24  | 192.168.11.254 | 10     |
| 20      | 行政部        | 192.168.21.0/24  | 192.168.21.254 | 20     |
| 30      | 管理部        | 192.168.31.0/24  | 192.168.31.254 | 30     |
| 40      | 人力资源部    | 192.168.41.0/24  | 192.168.41.254 | 40     |
| 50      | 市场部        | 192.168.51.0/24  | 192.168.51.254 | 50     |
| 60      | 销售部        | 192.168.61.0/24  | 192.168.61.254 | 60     |
| 70      | 项目部        | 192.168.71.0/24  | 192.168.71.254 | 70     |
| 80      | 生产部        | 192.168.81.0/24  | 192.168.81.254 | 80     |
| 100     | 食堂区域(无线)| 192.168.101.0/24 | 192.168.101.254 | 100    |
| 101     | 休闲区域(无线)| 192.168.102.0/24 | 192.168.102.254 | 101    |
| 150     | 服务器        | 192.168.151.0/24 | 192.168.151.254 | -      |

## ⚙️ 关键技术实现与代码片段

### 1. HSRP - 核心层网关冗余

配置双核心交换机，为每个VLAN提供冗余网关。优先级为120的设备成为Active，并启用抢占。

```cisco
! Core-Switch-1 (Active Device for VLAN 10)
interface Vlan10
 ip address 192.168.11.253 255.255.255.0
 standby 10 ip 192.168.11.254
 standby 10 priority 120
 standby 10 preempt
```
### 2. OSPF - 全网动态路由
在所有三层设备上启用OSPF Area 0，宣告直连网段，实现全网路由可达。
```
cisco
router ospf 1
 network 192.168.11.0 0.0.0.255 area 0
 network 192.168.21.0 0.0.0.255 area 0
 network 192.168.31.0 0.0.0.255 area 0
 network 192.168.41.0 0.0.0.255 area 0
 network 192.168.51.0 0.0.0.255 area 0
 network 192.168.61.0 0.0.0.255 area 0
 network 192.168.71.0 0.0.0.255 area 0
 network 192.168.81.0 0.0.0.255 area 0
 network 192.168.101.0 0.0.0.255 area 0
 network 192.168.102.0 0.0.0.255 area 0
 network 192.168.151.0 0.0.0.255 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/1
 no passive-interface GigabitEthernet0/2
```
### 3. ACL - 安全访问控制
使用扩展ACL实现精细化的访问控制策略。

策略A: 只允许财务(Vlan10)、行政(Vlan20)访问服务器(Vlan150)

策略B: 禁止生产部(Vlan80)访问外部互联网
```
cisco
! 允许特定部门访问服务器，拒绝其他所有IP流量
access-list 110 permit ip 192.168.11.0 0.0.0.255 192.168.151.0 0.0.0.255
access-list 110 permit ip 192.168.21.0 0.0.0.255 192.168.151.0 0.0.0.255
access-list 110 permit ip 192.168.31.0 0.0.0.255 192.168.151.0 0.0.0.255
access-list 110 permit ip 192.168.41.0 0.0.0.255 192.168.151.0 0.0.0.255
access-list 110 deny ip any any
!
interface Vlan150
 ip access-group 110 in
```
```
! 禁止生产部访问外网
access-list 120 deny ip 192.168.81.0 0.0.0.255 any
access-list 120 permit ip any any
!
interface GigabitEthernet0/1
 ip access-group 120 out
```
### 4. NAT - 互联网出口
在边界路由器上配置PAT，使内部私有IP地址通过一个公网IP访问互联网。
```
cisco
ip nat inside source list 1 interface GigabitEthernet0/1 overload
!
! ACL 1 定义允许转换的私有地址范围
access-list 1 permit 192.168.0.0 0.0.255.255
access-list 1 deny any
!
interface GigabitEthernet0/1
 ip nat outside
!
interface GigabitEthernet0/0
 ip nat inside
```
### 5. DHCP - 自动地址分配
在核心交换机上为每个VLAN配置DHCP地址池，实现终端即插即用。
```
cisco
ip dhcp excluded-address 192.168.11.254
ip dhcp pool Vlan10_Finance
 network 192.168.11.0 255.255.255.0
 default-router 192.168.11.254
 dns-server 8.8.8.8
 domain-name xiaoyi.com

ip dhcp excluded-address 192.168.21.254
ip dhcp pool Vlan20_Admin
 network 192.168.21.0 255.255.255.0
 default-router 192.168.21.254
 dns-server 8.8.8.8
 domain-name xiaoyi.com
```
### 6. 无线网络配置
```
cisco
! WLC配置
interface BVI1
 ip address 192.168.100.1 255.255.255.0

! AP配置
interface Dot11Radio0
 ssid Xiaoyi-Guest
 authentication open
 encryption vlan 100
 station-role root
```

## 如何打开与使用
### 环境要求:

Cisco Packet Tracer 8.2.0 或更高版本

Windows操作系统

### 快速开始:

1. 下载项目文件
git clone https://github.com/your-username/Xiaoyi-Enterprise-Network.git

2. 使用Packet Tracer打开项目文件
文件 -> 打开 -> 选择 Xiaoyi-Company-Network.pkt

### 测试功能:

进入"模拟模式"观察数据包流向

使用PC的命令行进行ping测试

通过show running-config查看设备配置

# 更多详情请查看“小易智慧公司详细说明”
