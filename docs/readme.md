# 服务治理

## 基础知识补充

https://www.processon.com/view/link/611fc4c85653bb6788db4039#map

opentrace spec：https://github.com/opentracing/specification/blob/master/specification.md#the-opentracing-data-model



对象内存查看工具：jol-core.jar



https://github.com/alibaba/transmittable-thread-local 解决异步线程池上下文传递问题

https://www.iteye.com/blog/nijiaben-1847212/ java agent 机制



## 灰度框架

### SDK ， 对业务代码有一定侵入， 但可提高性能

### agent ， 业务代码无侵入，有一定性能损耗

### mesh,  sidecar的方式， 无侵入， 引入三方数据面框架，性能无损耗、业务无侵入

## 初步方案

结合实际，业务场景；目前技术栈选择 java + SSM 为 agent 方式提供天然条件。

mesh sidecar 主流的方式是基于 istio 的方案，但是目前 istio 在生产落地过程遇到一些问题

如： 控制平面会在业务pod 中注入 sidecar 的配置，将当值api yaml 的膨胀问题

envoy 转发性能问题，由于在sidecar 需要做 诸如鉴权、授权、链路追踪等将影响到envoy 的转发性能



鉴于此，基于agent 的方式是比较好的选择

基于开源 skywalking 做字节码增强来完成灰度标的链路传递、灰度链路流量调度（如 rpc 灰度、MQ 灰度）

apisix 流量网关做 流量着色 和 流量切分



## 演进方案

**service mesh 方案** 

## 涉及技术领域

### RPC 

1.**线上流量debug**， 本地通过跳板机将本地服务注册到注册中心，希望线上流量满足路由规则后路由到本地服务，不满足路由规则的路由到线上实例。

### MQ

**全链路压测场景**    : 压测流量发送消息到影子Topic ，压测流量只订阅影子Topic

**流量隔离/全链路 灰度场景**   :使用相同的topic，线上流量订阅线上消息，隔离流量只订阅灰度消息



https://mp.weixin.qq.com/s/tmm7kdbwEmCEUKKzkEcD_A

### Database

**全链路压测场景**   :  压测流量数据库到影子表上

**高可用切流场景 **   : 禁止数据库操作；单元化下，流量没有单元标，禁止数据库操作。

### Redis

全链路压测场景下，压测流量缓存落库到影子KEY上

高可用切流场景下，禁止缓存操作；单元化下，流量没有单元标，禁止缓存操作。

### 分布式任务调度

任务调度，灰度环境提交的任务，被调度到灰度环境的机器上执行。

### 前端

不同客户看到的页面信息不一致

### 可观测性

通过可观测性监控流量走向， 查看流量逃逸情况。



## 灰度过程

### 服务发布 - RPC 灰度注册，MQ 灰度Topic 

### 流量染色

1. 灰度流量入口：gw 给在流量入口根据灰度规则进行流量着色
2. 灰度链路应用：灰度链路中应用应知道自身是否属于灰度应用，进行注册；此时存在  灰度组/灰度标

### 灰度标全链路传递

### 灰度流量 上下游处理

### 灰度流量终结





业界可参考方案：

https://blog.csdn.net/weixin_43318367/article/details/113053445



## 灰度发布

### 给予K8s 金丝雀发布

局限：K8s Service只在TCP侧面解决负载均衡问题，并不对请求响应的消息内容做任何解析和识别

### 业务编码实现

### Spring Cloud灰度发布

### Istio灰度发布



## 故障定位

### 全链路追踪

### 分布式日志

### 告警&监控

## CI

### 统一开发框架

### RPC 框架

### 服务治理

#### 降级&熔断

### 数据库中间件

### 消息中间件





## skywalking

### opentracing 规范

#### span理解

1. span tag
2. span log
3. span bagger

### SpanContext

处理跨进程链路追踪

SpanContext#inject

SpanContext#extract



ContextCarrier格式：text map 和 binary map



### skywaling 跨进程trace 核心逻辑





## skywalking 二次开发灰度方案

### 灰度标传递

基于 skywalking span （ExitSpan、NoopSpan） TraceCarrier 携带灰度标

### 灰度链路调度逻辑

#### RPC 灰度链路

基于字节码增强，修改RPC invoke 逻辑、结合注册中心元数据注册灰度服务

#### MQ消息灰度

MQ Header 中的灰度标， 字节码增强 MQ producer consumer 逻辑



## Skywalking core 解析

```java
public interface AbstractTracerContext {
  /*
  	处理跨进程传递
  */
  void inject(ContextCarrier carrier);
  
  // 关联跨进程的两个 segement
  void extract(ContextCarrier carrier);
  
  //处理跨线程
  ContextSnapshot capture();
  
  void continued(ContextSnapshot snapshot);
}
```



