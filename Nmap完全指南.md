# Nmap 完全指南

> **目标**：掌握 Nmap 端口扫描工具，能独立完成主机发现、端口扫描、服务识别、操作系统探测，面试时能口述核心参数和扫描策略。

---

## 一、Nmap 简介

### 1.1 什么是 Nmap

**Nmap**（Network Mapper）是网络安全领域最著名的开源端口扫描工具，支持：

- 主机发现（Ping 扫描）
- 端口扫描（TCP/UDP）
- 服务版本识别
- 操作系统探测
- 漏洞扫描（NSE 脚本）
- 防火墙/IDS 规避

### 1.2 为什么面试要问 Nmap

| 考察点 | 面试官想知道什么 |
|--------|---------------|
| 工具使用 | 你是否会用专业工具进行信息收集 |
| 网络基础 | 你理解 TCP/UDP、三次握手等原理 |
| 扫描策略 | 不同场景下选择什么扫描方式 |
| 隐蔽意识 | 知道如何规避防火墙和 IDS |

### 1.3 Nmap vs 其他扫描工具

| 对比项 | Nmap | Masscan | Zmap |
|--------|------|---------|------|
| 速度 | 中 | 极快 | 极快 |
| 功能 | 全面（版本/OS/脚本） | 简单（仅端口） | 简单（仅端口） |
| 准确度 | 高 | 中 | 中 |
| 隐蔽性 | 可配置 | 低 | 低 |
| 面试要求 | 必须会 | 了解即可 | 了解即可 |
| 实际工作 | 详细扫描用 Nmap | 快速发现用 Masscan | 互联网级扫描 |

**面试一句话**："Masscan/Zmap 快速发现开放端口，Nmap 详细识别服务和漏洞，两者结合效率最高。"

---

## 二、Nmap 安装与基础配置

### 2.1 安装方式

#### 方式一：Kali Linux 自带（推荐）

```bash
# Kali 已预装
nmap --version
```

#### 方式二：Linux 包管理器

```bash
# Ubuntu/Debian
sudo apt-get update && sudo apt-get install nmap

# CentOS/RHEL
sudo yum install nmap
```

#### 方式三：macOS

```bash
brew install nmap
```

#### 方式四：Windows

下载安装包：https://nmap.org/download.html

### 2.2 基础参数速查表

| 参数 | 全称 | 作用 | 使用频率 |
|------|------|------|---------|
| `-sP` | `-sn` | Ping 扫描，主机发现 | ⭐⭐⭐⭐⭐ |
| `-sS` | `--syn` | TCP SYN 半开放扫描 | ⭐⭐⭐⭐⭐ |
| `-sT` | `--tcp` | TCP Connect 全连接扫描 | ⭐⭐⭐⭐ |
| `-sU` | `--udp` | UDP 扫描 | ⭐⭐⭐⭐ |
| `-sV` | `--version` | 服务版本识别 | ⭐⭐⭐⭐⭐ |
| `-O` | `--osscan` | 操作系统探测 | ⭐⭐⭐⭐ |
| `-A` | `--all` | 全面扫描（-sV -O -sC） | ⭐⭐⭐⭐⭐ |
| `-p` | `--port` | 指定端口范围 | ⭐⭐⭐⭐⭐ |
| `-T` | `--timing` | 扫描速度（0-5） | ⭐⭐⭐⭐ |
| `-v` | `--verbose` | 详细输出 | ⭐⭐⭐⭐ |
| `-oN` | `--output-normal` | 保存为文本格式 | ⭐⭐⭐⭐ |
| `-oX` | `--output-xml` | 保存为 XML 格式 | ⭐⭐⭐ |
| `--script` | `--script` | 使用 NSE 脚本 | ⭐⭐⭐⭐ |
| `-Pn` | `--noping` | 跳过主机发现 | ⭐⭐⭐⭐ |
| `-f` | `--frag` | 分片扫描，绕过防火墙 | ⭐⭐⭐ |

---

## 三、Nmap 实操

### 3.1 环境准备

**目标**：扫描本地网段或靶机环境

**命令**：
```bash
# 查看本机 IP
ifconfig
# 或
ip addr
```

![78455575583](Nmap完全指南.assets/1784555755830.png)

### 3.2 主机发现（Ping 扫描）

```bash
nmap -sP 192.168.133.129
```

**参数说明**：
- `-sP`：Ping 扫描，只发现在线主机，不扫描端口
- `192.168.133.0/24`：扫描整个 C 段网段

**预期输出**：

```
Nmap scan report for 192.168.133.1
Host is up (0.00032s latency).
MAC Address: 00:50:56:C0:00:08 (VMware)

Nmap scan report for 192.168.133.100
Host is up (0.00045s latency).
MAC Address: 00:0C:29:XX:XX:XX (VMware)
```

![78455632200](Nmap完全指南.assets/1784556322005.png)

### 3.3 端口扫描

```bash
nmap 192.168.133.129
```

**参数说明**：
- 默认扫描 Top 1000 端口
- 使用 SYN 半开放扫描（需要 root）

**预期输出**：
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
```

![78455598773](Nmap完全指南.assets/1784555987730.png)

### 3.4 指定端口扫描

```bash
# 扫描指定端口
nmap -p 22,80,443,3306,8080 192.168.133.129

# 扫描全部端口（1-65535）
nmap -p- 192.168.133.129

# 扫描端口范围
nmap -p 1-1000 192.168.133.129
```

**参数说明**：
- `-p`：指定端口，多个用逗号分隔，范围用 `-`
- `-p-`：扫描全部 65535 个端口

![78455613207](Nmap完全指南.assets/1784556132070.png)

![78455616636](Nmap完全指南.assets/1784556166367.png)![78455619119](Nmap完全指南.assets/1784556191199.png)

### 3.5 服务版本识别

```bash
nmap -sV 192.168.133.100
```

**参数说明**：
- `-sV`：探测开放端口上运行的服务版本

**预期输出**：
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
3306/tcp open  mysql   MySQL 5.7.26
```

![78455623031](Nmap完全指南.assets/1784556230316.png)

### 3.6 操作系统探测

```bash
nmap -O 192.168.133.129
```

**参数说明**：
- `-O`：通过 TCP/IP 指纹探测操作系统类型

**预期输出**：
```
OS details: Microsoft Windows 10 1607 - 1809
```

![78455636443](Nmap完全指南.assets/1784556364432.png)

### 3.7 全面扫描（推荐）

```bash
nmap -A 192.168.133.129
```

**参数说明**：
- `-A`：全面扫描，等价于 `-sS -sV -O --script=default`
- 包含：SYN 扫描 + 版本识别 + OS 探测 + 默认 NSE 脚本

![78455641222](Nmap完全指南.assets/1784556412222.png)

### 3.8 NSE 脚本扫描

```bash
# 查看所有脚本
ls /usr/share/nmap/scripts/ | head -20

# 使用漏洞扫描脚本
nmap --script=vuln 192.168.133.129

# 使用特定脚本（如 HTTP 标题）
nmap --script=http-title 192.168.133.129

# 使用多个脚本
nmap --script=http-title,http-headers 192.168.133.129
```

**参数说明**：
- `--script`：调用 Nmap Scripting Engine 脚本
- `vuln`：漏洞扫描脚本集合

![78455650291](Nmap完全指南.assets/1784556502918.png)

![78455662772](Nmap完全指南.assets/1784556627720.png)

![78455681292](Nmap完全指南.assets/1784556812922.png)

![78455683894](Nmap完全指南.assets/1784556838947.png)

### 3.9 扫描速度控制

```bash
# 极速扫描（可能丢包）
nmap -T4 192.168.133.129

# 慢速扫描（规避 IDS）
nmap -T1 -f 192.168.133.129
```

| 级别 | 参数 | 说明 |
|------|------|------|
| 0 | `-T0` | 极慢（IDS 规避） |
| 1 | `-T1` | 慢速 |
| 2 | `-T2` | 正常 |
| 3 | `-T3` | 快速（默认） |
| 4 | `-T4` | 极速 |
| 5 | `-T5` | 疯狂（可能丢包） |

![78455689069](Nmap完全指南.assets/1784556890698.png)

### 3.10 保存扫描结果

```bash
# 保存为文本
nmap -oN result.txt 192.168.133.129

# 保存为 XML
nmap -oX result.xml 192.168.133.129

# 保存为所有格式
nmap -oA result 192.168.133.129
# 生成 result.nmap, result.xml, result.gnmap
```

**参数说明**：
- `-oN`：普通文本格式
- `-oX`：XML 格式（可被其他工具导入）
- `-oA`：同时生成三种格式

![78455811671](Nmap完全指南.assets/1784558116712.png)

![78455816959](Nmap完全指南.assets/1784558169591.png)

---

## 四、Nmap 高级用法

### 4.1 TCP SYN vs TCP Connect 扫描

```bash
# SYN 半开放扫描（需要 root，速度快，隐蔽）
sudo nmap -sS 192.168.133.129

# TCP Connect 全连接扫描（不需要 root，日志留痕）
nmap -sT 192.168.133.129
```

| 对比项 | SYN 扫描 (-sS) | Connect 扫描 (-sT) |
|--------|---------------|-------------------|
| 权限 | 需要 root | 不需要 root |
| 速度 | 快 | 较慢 |
| 隐蔽性 | 高（不完成握手） | 低（完成三次握手） |
| 日志 | 目标不记录 | 目标会记录 |

![78455778234](Nmap完全指南.assets/1784557782343.png)

![78455781148](Nmap完全指南.assets/1784557811484.png)

### 4.2 UDP 扫描

```bash
nmap -sU 192.168.133.129
```

**特点**：
- UDP 无状态，扫描速度慢
- 常用于发现 DNS、SNMP 等服务

![78455790647](Nmap完全指南.assets/1784557906477.png)

### 4.3 跳过主机发现（-Pn）

```bash
nmap -Pn 192.168.133.129
```

**场景**：
- 目标禁 Ping
- 防火墙过滤 ICMP
- 直接扫描端口，不先探测主机是否在线


- ![78455794059](Nmap完全指南.assets/1784557940594.png)

### 4.4 分片扫描绕过防火墙

```bash
nmap -f 192.168.133.129
```

**原理**：
- 将数据包分片发送，绕过简单的包过滤防火墙
- 防火墙可能只检查第一个分片


- ![78455797098](Nmap完全指南.assets/1784557970981.png)

---

## 五、Nmap 与 SQLMap 结合

### 5.1 信息收集 → 漏洞利用流程

```bash
# 第一步：Nmap 发现开放端口和服务
nmap -A 192.168.133.129

# 第二步：发现 80 端口有 Web 服务
# 第三步：访问网站，寻找注入点
# 第四步：SQLMap 自动化注入
sqlmap -u "http://192.168.133.129/page.php?id=1" --batch
```

![78455802775](Nmap完全指南.assets/1784558027759.png)



![78455804880](Nmap完全指南.assets/1784558048806.png)

---

## 六、Nmap 与 Masscan 对比总结

### 6.1 面试口述框架

**Q：Nmap 和 Masscan 怎么选？**

> "Masscan 异步扫描速度极快，适合快速发现全网开放端口（如 5 分钟扫完一个 B 段）。Nmap 功能全面，支持服务识别、OS 探测、NSE 脚本，适合对目标进行详细分析。实际工作中先用 Masscan 快速发现，再用 Nmap 详细扫描。"

**Q：Nmap 的 -sS 和 -sT 有什么区别？**

> "-sS 是 SYN 半开放扫描，只发送 SYN 包，收到 SYN+ACK 后发送 RST 不完成三次握手，速度快且隐蔽，但需要 root 权限。-sT 是 TCP Connect 全连接扫描，完成完整的三次握手，不需要 root，但会在目标日志中留下连接记录。"

**Q：Nmap 怎么绕过防火墙？**

> "多种方法：1. -f 分片扫描，将数据包分片发送；2. -D 使用诱饵主机隐藏真实 IP；3. --source-port 指定常见端口（如 53）作为源端口；4. -T0/-T1 慢速扫描规避频率检测；5. -Pn 跳过主机发现直接扫描端口。"

### 6.2 核心参数速记口诀

```
-sP 发现主机，-sS 隐蔽扫描
-sV 看版本，-O 猜系统
-A 全扫一遍，-p 指定端口
-T 控速度，--script 跑脚本
-oN 存文本，-Pn 跳过Ping
-f 分片绕防火墙
```

---

## 七、今日产出清单

| 产出 | 状态 |
|------|------|
| 主机发现 | `-sP` 扫描网段 |
| 端口扫描 | 默认 Top 1000 |
| 指定端口 | `-p 22,80,3306` |
| 全部端口 | `-p-` |
| 服务版本 | `-sV` |
| 操作系统 | `-O` |
| 全面扫描 | `-A` |
| NSE 脚本 | `--script=vuln` |
| 速度控制 | `-T0` 到 `-T5` |
| 保存结果 | `-oN / -oX / -oA` |
| SYN vs Connect | `-sS` vs `-sT` |
| UDP 扫描 | `-sU` |
| 跳过主机发现 | `-Pn` |
| 分片绕过 | `-f` |

---

## 八、面试高频题

**Q1：Nmap 的 -sS 和 -sT 有什么区别？**

> "-sS 是 SYN 半开放扫描，只发送 SYN 包，不完成三次握手，速度快且隐蔽，但需要 root 权限。-sT 是 TCP Connect 全连接扫描，完成完整的三次握手，不需要 root，但会在目标日志中留下记录。"

**Q2：Nmap 怎么绕过防火墙？**

> "五种方法：1. -f 分片扫描；2. -D 诱饵主机；3. --source-port 指定常见端口；4. -T0/-T1 慢速扫描；5. -Pn 跳过主机发现。"

**Q3：Nmap 的 -A 参数做了什么？**

> "-A 是全面扫描，等价于 -sS -sV -O --script=default，即 SYN 扫描 + 服务版本识别 + 操作系统探测 + 默认 NSE 脚本扫描。会消耗较长时间，但信息最全面。"

**Q4：Nmap 和 Masscan 有什么区别？**

> "Nmap 功能全面，支持服务识别、OS 探测、NSE 脚本，但扫描速度较慢。Masscan 是异步扫描器，速度极快（号称 5 分钟扫完全网），但功能简单，只做端口扫描。实际工作中先用 Masscan 快速发现开放端口，再用 Nmap 详细扫描。"

**Q5：Nmap 扫描时怎么判断端口是否开放？**

> "TCP 扫描中，收到 SYN+ACK 表示端口开放，收到 RST 表示端口关闭，无响应表示被过滤。UDP 扫描中，收到 UDP 响应表示开放，收到 ICMP 不可达表示关闭，无响应表示开放或被过滤。"

**Q6：Nmap 的 --script 能做什么？**

> "NSE（Nmap Scripting Engine）脚本可以扩展 Nmap 的功能，包括：漏洞扫描（--script=vuln）、服务信息收集（http-title、http-headers）、暴力破解（ssh-brute）、信息泄露检测等。脚本存放在 /usr/share/nmap/scripts/ 目录。"

---

## 九、参考资源

- Nmap 官方文档：https://nmap.org/book/
- Nmap 脚本库：https://nmap.org/nsedoc/
- Nmap 下载：https://nmap.org/download.html
- Masscan：https://github.com/robertdavidgraham/masscan

---

> **笔记整理日期**：2026-07-20

