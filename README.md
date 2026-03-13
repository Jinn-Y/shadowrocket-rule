# Shadowrocket Rules

一个用于 `Shadowrocket` 的个人分流规则仓库，核心配置文件是 [lazy.conf](./lazy.conf)。

这份规则的目标是：

- 保持国内常用服务直连
- 让海外常用服务按地区或代理策略分流
- 提供适合日常使用的节点分组
- 支持通过订阅地址自动更新配置

## 项目内容

仓库目前主要包含以下文件：

- [lazy.conf](./lazy.conf)：主配置文件
- [README.md](./README.md)：项目说明

`lazy.conf` 中主要包括：

- `[General]`：DNS、IPv6、TUN、跳过代理网段等基础设置
- `[Proxy Group]`：按地区划分的节点组，例如香港、台湾、日本、新加坡、韩国、美国
- `[Rule]`：按服务分流的规则集
- `[Host]`、`[URL Rewrite]`、`[MITM]`：为后续扩展预留

## 分流思路

当前配置大致采用以下策略：

- 国内服务走 `DIRECT`
- 中国大陆 IP 走 `GEOIP,CN,DIRECT`
- 海外通用服务走 `PROXY`
- 部分服务指定固定地区节点

示例：

- `OpenAI`、`Google`、`TikTok`、`PayPal` 默认走美国节点
- `Netflix` 默认走日本节点
- `Apple`、`Bilibili`、`微信`、`百度`、`知乎` 等走直连
- 未命中的流量最终走 `FINAL,PROXY`

## 使用方式

### 1. 直接导入配置

将 [lazy.conf](./lazy.conf) 导入 `Shadowrocket` 使用。

### 2. 通过远程地址更新

配置文件中已包含更新地址：

```text
https://gh-proxy.com/https://raw.githubusercontent.com/Jinn-Y/shadowrocket-rule/refs/heads/main/lazy.conf
```

如果你是此仓库维护者，可以直接将它作为订阅更新地址使用。

## 适用场景

这份规则更适合以下需求：

- 日常上网分流
- 海外服务按地区节点访问
- 减少手动切换代理策略
- 结合现有订阅节点快速使用

## 参考项目

本项目规则设计参考了以下项目：

- [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)

## 说明

- 这是个人使用规则，策略偏好以维护者自己的使用习惯为主
- 远程规则集内容可能随上游项目变化而变化
- 如需调整分流策略，可直接编辑 [lazy.conf](./lazy.conf)
