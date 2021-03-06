### 流量染色切换

##### 问题

- IOS版本升级的时候，会有一段时间有问题，具体多少时间取决于后端升级的时间
- 改动比较大代码的时候，回滚时间较长
- 改动比较大代码的时候，部署时间长



##### 方案说明

代码需要部署两套，分别命名为红和黄，红和黄都有可能部署升级版本。



##### 具体做法

在前端对流量进行着色，具体需要客户端通过版本信息请求后端接口，后端接口返回红或者黄。假设现在所有版本的流量都是黄色的，比如...3.13,3.14,现在需要升级3.15。

1. All(<=3.14)->Yellow
2. Old(<=3.14)->Yellow,New(=3.15)->Red
3. All(<=3.15)=>Red

通过以上步骤之后，升级完成，所有流量其实都已经导向了红。之后再有版本升级，其实也是从红到黄或者黄到红之间的过程。

Step1图示

```
                  +------------------+
                  |   Client(all)    |
                  +------------------+
                            |
                           Y|
                            |
                            |
                 +----------v----------+
                 |       Gateway       |
                 |                     |
                 +---------------------+
                            |
                            |
                           Y|
+---------------+           |           +---------------+
|    Yellow     |<----------+           |      Red      |
+---------------+                       +---------------+
```

Step2图示

```
+------------------+                     +------------------+
|   Client(old)    |                     |   Client(new)    |
+------------------+                     +---------+--------+
          |                                        |
          |                                        |
          |                                       R|
         Y|                                        |
          |                                        |
          |                                        |
          |        +---------------------+         |
          |        |       Gateway       |         |
          +------->|                     |<--------+
                   +---+-------------+---+
                       |             |
                       |            R|
                      Y|             |
  +---------------+    |             |    +---------------+
  |    Yellow     |<---+             +--->|      Red      |
  +---------------+                       +---------------+
```

Step3图示

```
                   +-----------------+
                   |   Client(all)   |
                   +-----------------+
                            |
                           R|
                            |
                            |
                 +----------v----------+
                 |       Gateway       |
                 |                     |
                 +---------------------+
                            |
                            |
                           R|
+---------------+           |           +---------------+
|    Yellow     |           +---------->|      Red      |
+---------------+                       +---------------+
```

Gateway上需要同时转发两套环境，分别是红upstream和黄upstream，并且根据Client Header某个域，把流量转发到不同的upstream

后端提供一个接口，根据前端传入的版本号和后台配置的版本切换信息，输出红或者黄标记。



##### 总结

该方案能同时解决开头提出来的问题1和问题2，也能一定程度解决问题3，重点就是要设计版本切换的逻辑。

