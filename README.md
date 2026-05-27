# nftables 模块化配置说明

## 概述
本配置将 nftables 规则拆分为独立模块，所有可变访问数据（IP、端口、映射）集中在 `sets.nft` 中管理，规则逻辑固化在 `chain-*.nft` 文件中。适用于以下两种场景：
1. **本机服务器**：处理发往本机的请求（如 Web 服务、SSH）。
2. **代理/网关**：处理经本机转发的请求（如 NAT 转发、正向代理）。
3. **启动服务**: systemctl start nftables.service

## 快速开始
```bash
## 检查并优化规则配置
sudo nft -c -o -f main.nft
## 加载配置规则
sudo nft -f main.nft

## 查看规则
sudo nft list ruleset
```

## 自定义指南
- **增加新的允许源 IP**：在 `sets.nft` 的 `local_allow_sources` 或 `forward_sources` 集合中添加元素。
- **开放新端口**：向 `local_tcp_services` 或 `forward_ports` 等集合添加端口号。
- **修改转发目标**：更新 `dnat_map` 字典中的键值对（需配合 nat 表规则）。
- **禁用某项允许**：从对应集合中删除元素即可。

## 注意事项
- 默认策略为 `input` 和 `forward` 丢弃，`output` 接受，请确保 `local_allow_sources` 包含您的管理地址。
- 日志记录前缀为 `INPUT_DROP` 和 `FORWARD_DROP`，可通过 `dmesg` 或 `journalctl -k` 查看。
- 如需支持 IPv6，请为相关集合增加 `type ipv6_addr` 并添加对应规则（可仿照 IPv4 编写）。
