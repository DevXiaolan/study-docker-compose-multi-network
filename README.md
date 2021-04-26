## docker-compose 网络设置研究

### 网络关系定义

![](https://lanhaooss.oss-cn-shenzhen.aliyuncs.com/images/333/333-1.png)

如图所示，我们在 `docker-compose.yml` 文件了里定义了 4 个服务（ redis实例 ）。其中：

- u1 声明在 n1 网络下
- u2 声明在 n1、n2 网络下（多网络模型）
- u3 不做特别声明
- u4 声明在 n2 网络下

### 测试方式

通过 `docker-compose up` 启动全部服务，然后进入到特定的容器内部（如 u1），通过 `redis-cli -h u2` 方式测试是否能访问`u2`服务，其他类推。

### 结论与现象

在 `docker-compose up` 阶段，可以看到终端输出

```
Creating network "docker-compose-multi-network_n1" with driver "bridge"
Creating network "docker-compose-multi-network_n2" with driver "bridge"
Creating network "docker-compose-multi-network_default" with the default driver
```
说明了：

> 有被声明用到的 n1、n2 网络，会被创建出来。
>
> 如果有服务没有声明指定网络(u3), 就会创建一个 default 网络

通过 `redis-cli` 测试得出

- 声明了在同一网络下的容器可以相互访问

  > u1与u2, u2与u4，分别可以互通
- 没有声明网络的，会分配到 default网络下

  > u3 与其他都不互通
- 有声明网络的，不会分到default下

  > u1、u2、u4都不能访问 u3
- 声明了多个网络的情况下，可以与各自网络内的容器访问 

  > u2可以分别与 u1、u4互通，但u1与u4不互通 