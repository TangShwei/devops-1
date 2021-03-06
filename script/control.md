# 服务起停脚本
当前很多启动停止脚本存在的问题：启动start只是简单的port检查， 停止stop也是野蛮的killall 操作，传参也是多样性，同时也缺乏有效的判断检查服务状态的接口。control脚本可以统一化，加入标准的启停接口，增加启停服务的健壮性。

## 规范
- 最小命令集合：start/stop/status
- 进程启动前必须对依赖进行检查
- 部署路径以及启动脚本：$/bin/control
- 进程在15min内coredump次数大于等于5次，supervise需要退出
- 关闭进程应该对进程名进行一一匹配，不应该仅仅检查pid
- 合理设置进程启动和关闭的超时时间
- 非正常状态应该进行捕获并返回有效的排查内容
- 应该在进程的bin目录下启动服务

## 程序启动前的自检内容
在程序启动前，需要检查程序所依赖的环境是否存在并满足运行的最低需求，包括但不限于以下内容：
- 操作系统，内核版本，内核模块
- 系统参数是否满足需求，如网卡多队列，超线程等
- 编译器如Java，Python，GCC的版本以及部署路径
- 权限如数据库授权，上下游授权，第三方依赖授权，网络是否可达
- 文件以及目录是否存在，数据是否存在
- 资源是否满足需求，如分配的CPU，MEM，FD，端口等


## supervise雪崩防范
程序在短时间内coredump，如果supervise具备拉起服务的能力，那么supervise应该设置最大的自动拉起次数和时间间隔，具体操作参照如下建议:
- supervise启动进程前，需要按照/proc/sys/kennel/core_pattern设置的路径进行core文件扫描
- 如果在15min内相关进程的core文件数量大于5，则supervise需要退出，避免再次重启引发服务雪崩
- 鉴于core文件的体积大小不同，因此supervise不应该承担core文件的清理工作，避免执行耗时较长导致服务启动出现问题

## stop功能
- 首先应该执行程序推荐的stop方式，并确保stop命令具备超时的功能
- 执行完毕后，应该通过sleep的方式检查并确认进程关闭成功
- 如果正常方式下未关闭进程，则需要重试至多3次
- 如果重试依然未生效，则可以采用kill的方式进行
- stop失败应该提供有效的输出信息便于快速定位
- 对进程的关闭需要严格进行一一匹配，禁止批量关闭进程，同时需要对关闭进程的数量进行判断，避免误杀
