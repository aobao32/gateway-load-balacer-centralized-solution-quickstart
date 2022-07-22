# gateway-load-balacer-centralized-solution-quickstart
GWLB Quickstart CloudFormation Template

## 一、架构说明

本方案构建一个集中式的网络检测方案，实现多个VPC的东西方向（Inter-VPC）、南北方向（Egress）的流量深度检测。

主要组件如下：

- NGFW：GWLB需要搭配Marketplaces启动的第三方商用防火墙，需要防火墙参加支持Geneve协议，本Quickstart中，使用Geneve-proxy模拟防火墙
- GWLB Endpoint：VPC的流量入口，作为路由表的下一跳接受流量
- GWLB：将流量分发给Target Group，目标组中每个EC2是接受Geneve协议封装后发送过来的原始流量，拆包检查
- NAT Gateway：本架构中所有对外流量的统一出口，所有VPC访问外部互联网访问均汇集到此后统一出口
- ELB：本架构中所有外部进入流量的统一入口
- VPC1：部署以上所有组件
- VPC2/3/4：部署应用程序（业务软件）
- Transit Gateway：VPC互联，且需要打开Appliance Mode模式支持跨AZ访问（参考下文配置）

网络流向：

- 东西流量：部署业务应用的VPC的默认路由的下一跳是TGW，到达TGW的ENI后送入GWLB Endpoint进行检测，检测后返回到另一个VPC
- 南北流量：从VPC出向去往Internet的流量，经过NAT转换后到达互联网，数据按原路径回包

总体架构图：

![总体架构](https://d51vuyprlknbq.cloudfront.net/GWLB/GWLB-quickstart-architecture-diagram.png)

## 二、启动模版使用说明

使用方法如下：

- 下载所有模版到S3存储桶内，包含主模版和所有Nested嵌套模版
- 将文件设置为Public可访问，包含主模版和所有Nested嵌套模版，获取他们公开后的URL
- 替换主模版 `CN-GWLB-TGW.yml` 文件中，各Nested嵌套模版的URL网址为自己的存储桶网址
- 进入Cloudformation界面，选择输入模版的S3地址，填入主模版 1CN-GWLB-TGW.yml` 的S3上公开的地址
- 填写Cloudformation参数，除模版名称和EC2 Keypair的名字需要指定外，其余可以默认，完成创建

## 三、模版启动后的Post-configuration

本Quickstart使用的是Geneve Proxy模拟防火墙。在模版启动后，在VPC1中可以看到带有标签名为 `AccessVPC-VirtualAppliance1` 和 `AccessVPC-VirtualAppliance2` 的
两个节点，这两个节点运行Amazon Linux 2，且已经下载了geneve proxy到 `/root/geneve-proxy` 目录。

配置方法是：

- 编辑 `/root/geneve-proxy/config.yml` 配置文件，编辑 `blocked_transport_protocols` 调整要拦截的访问，编辑 `allowed_transport_protocols` 允许访问，编辑完成保存退出
- 执行命令 `nohup python3 main.py &` 启动程序并排入后台
- 执行命令 `tcpdump -nvv 'port 6081'` 查看流量

以上配置过程可参考[这个](https://d5ubqqttnawof.cloudfront.net/video/GWLB-02-QuickstartDemo.mp4)视频（本视频有解说注意播放音量）。

## 四、开启Transit Gateway的跨AZ访问

Transit Gateway 默认会将流量保持在本AZ内，当结合GWLB时候，Inter-VPC的跨AZ访问会出现无法访问的问题，此时需要对GWLB所在的VPC的Transit Gateway Attachment打开Appliance Mode。其他VPC不需要变更此配置。参数如下：

```
aws ec2 modify-transit-gateway-vpc-attachment --transit-gateway-attachment-id tgw-attach-0253EXAMPLE --options ApplianceModeSupport=enable
```

注意本方法目前在Cloudformation接口无法调用，因此需要启动Quickstart模版完成后，通过执行上述CLI命令手工变更配置。

参考[这篇](https://aws.amazon.com/cn/blogs/networking-and-content-delivery/centralized-inspection-architecture-with-aws-gateway-load-balancer-and-aws-transit-gateway/ 
)官方博客。配置过程可参考[这个](https://d5ubqqttnawof.cloudfront.net/video/GWLB-03-TGWApplianceMode.mp4)视频（本视频有解说注意播放音量）。