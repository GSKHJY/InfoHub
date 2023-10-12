# Nginx

## Nginx 简介

Nginx 是一款轻量级的 Web 服务器 / 反向代理服务器 / 电子邮件（IMAP/POP3）代理服务器，主要的优点是：

1. 支持高并发连接，尤其是静态界面，官方测试 Nginx 能够支撑5万并发连接；
2. 内存占用极低；
3. 配置简单，使用灵活，可以基于自身需要增强其功能，同时支持自定义模块的开发使用灵活：可以根据需要，配置不同的负载均衡模式，URL 地址重写等功能；
4. 稳定性高，在进行反向代理时，宕机的概率很低；
5. 支持热部署，应用启动重载非常迅速。

## Nginx 在架构体系中的作用

- 网关：面向客户的总入口，可以拦截客户端所有请求，对该请求进行权限控制、负载均衡、日志管理、接口调用监控等；
- 虚拟主机：将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供 www 服务，这样就可以实现一台主机对外提供多个 web 服务，如 VPS 虚拟服务器；
- 路由：正向代理 / 反向代理，可用于解决跨域问题，同时配置内容和 API 接口等更加方便；
- 静态服务器：HTML、图片等静态资源；
- 负载集群：提供负载均衡。

## Nginx 配置文件

### 配置分离

在生产环境或者开发环境中 Nginx 一般会代理多个虚拟主机，如果把所有的配置文件写在默认的nginx.conf 中，看起来会非常臃肿，因此建议将每一个虚拟文件单独放置一个文件夹，在nginx.conf中导入各个配置，代码如下：

```
http {
	# 省略中间配置

	# 引用该目录下以 .conf 文件结尾的配置
    include /etc/nginx/conf.d/*.conf;
}
```

### Nginx 配置

https://juejin.cn/post/6844903575210967048#heading-14

## Nginx 基础命令

启动 Nginx
/usr/local/nginx/sbin/nginx
关闭 Nginx，建议使用quit，等待工作进程处理完毕后关闭
/usr/local/nginx/sbin/nginx -s stop
或
/usr/local/nginx/sbin/nginx -s quit
重启 Nginx 
/usr/local/nginx/sbin/nginx -s reload
生产服务器上执行 nginx 相关命令需要加 sudo

检查配置文件是否正确
/usr/local/nginx/sbin/nginx -t

## Nginx 日志分割

https://juejin.cn/post/6844903763216433159

## Nginx DNS 缓存问题

运维遇坑记录(3)-Nginx缓存了DNS解析造成后端不通
https://segmentfault.com/a/1190000020475756
https://cloud.tencent.com/developer/article/1888426

## 参考资料

「查缺补漏」巩固你的Nginx知识体系
Nginx 热部署和日志切割，你学会了吗？

## 进阶学习

《实战Nginx取代Apache的高性能web服务器》

Nginx 文档：https://nginx.org/en/docs/
