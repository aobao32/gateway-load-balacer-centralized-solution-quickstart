# gateway-load-balacer-centralized-solution-quickstart
GWLB Quickstart CloudFormation Template

## 一、背景

本方案构建一个集中式的网络检测方案，实现多个业务VPC的东西、南北流量深度检测。主要组件如下：

- NGFW：本方案使用Marketplaces上启动第三方商用防火墙，实现流量深度检测、可视化和审计等功能。需要防火墙厂家支持Geneve协议。
- GWLB Endpoint：流量入口，作为路由表的下一跳，接受流量
- GWLB：将流量分发给一组EC2，每个EC2是从Marketplaces启动的商用防火墙的AMI
- 东西流量检测：业务VPC之间通过TGW转发，并送入GWLB Endpoint进行检测
- 南北流量检测：从VPC出向去往Internet的流量，经过NAT转换后到达互联网，数据按原路径回包

## 二、启动模版使用说明

待更新