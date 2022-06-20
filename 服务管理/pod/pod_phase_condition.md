如何查看和分析Kubernetes中pod的phase、conditions？它们有什么作用？
2022年5月21日
Contents [hide]

一Pod的phase
0 pod的phase作用
1pod有哪些phase
2 从哪儿查看pod的phase
3 如何查看pod的phase
二 pod的conditions
0 pod有了phase，为什么还要有conditions
1 pod的conditions的作用
2 pod的conditions分类
3 从哪儿查看pod的conditions
4 如何查看pod的conditions
三 参考
一Pod的phase
0 pod的phase作用
用于描述、查看、分析pod当前处于什么状态。

1pod有哪些phase
通常情况下，在pod的生命周期中，每个pod会处于5个不同的phase：pending，running，succeed，failed，unknown。同一时间，1个pod只能处于1个phase。

当pod刚被创建时，它处于pending这个phase，等待被调度；
如果pod中的一个或多个container处于运行状态时，那么pod就处于running phase；
如果pod中的container不是被设置为无限运行下去的情况下(比如执行定时任务或一次性任务)，且container运行结束，那么pod处于succeed phase；
反之，如果pod中的container不是被设置为无限运行下去的情况下(比如执行定时任务或一次性任务)，且container运行失败，那么pod处于failed phase；
如果pod所在node上的kubelet出现故障或意外，而停止向Kubernetes API server报告它所在node上的pod的状态时，那么此时该node上的pod就处于unknown phase；
2 从哪儿查看pod的phase
pod的yaml文件里

3 如何查看pod的phase
由于pod的phase字段位于pod的manifest中的Status部分，也就是说 ，我们可以从Kubernetes API server那里获取pod的yaml文件，然后从status字段中找到pod的phase。那么，我们就可以，通过kubectl get pod pod_name -o yaml|grep phase来查看pod的phase：

1
[root@master-node ~]# kubectl get pods
2
NAME                       READY   STATUS      RESTARTS   AGE
3
curl                       1/1     Running     0          6d9h
4
curl-with-ambassador       2/2     Running     0          28d
5
downward                   1/1     Running     0          28d
6
fortune-configmap-volume   2/2     Running     0          36d
7
fortune-https              2/2     Running     0          35d
8
my-job-jfhz9               0/1     Completed   0          6d9h
9
[root@master-node ~]# kubectl get pods curl -o yaml|grep phase
10
  phase: Running
11
[root@master-node ~]# kubectl get pods my-job-jfhz9 -o yaml|grep phase
12
  phase: Succeeded
13
[root@master-node ~]# 
从上，我们通过pod的yaml文件里获取了它们的phase。其中的my-job-jfhz9是一个job且已经执行完成。所以，它处于succeed的phase。

二 pod的conditions
0 pod有了phase，为什么还要有conditions
因为pod的phase比较简单的描述了pod处于哪个具体情况，但是没有明确说明具体原因。

1 pod的conditions的作用
用于描述1个pod当前是否处于哪个phase，以及处于该phase的原因。及作为一个辅助手段，详细的展示pod的状态信息，用于问题排查分析时提供更多依据。同一时间，1个pod可能处于多个conditions。

2 pod的conditions分类
通常分为4个conditions：PodScheduled，Initialized，ContainersReady，Ready。见名知意：

PodScheduled：意味着pod是否已经被调度到某个node；
Initialized:Pod的init containers是否全部完成；
ContainersReady：pod中的所有container是否全部就绪；但这并不意味着pod也ready；
Ready：pod是否就绪；只有pod中的所有container就绪，且pod的readiness probe也完成了，意味着pod可以对外提供服务了，才是ready状态。
3 从哪儿查看pod的conditions
pod的yaml文件。

4 如何查看pod的conditions
同样，由于pod的conditions源于yaml格式的manifest中的Status字段，我们可以从yaml文件里查看。

1
[root@master-node ~]# kubectl get pods curl -o yaml
2
apiVersion: v1
3
kind: Pod
4
...
5
status:
6
  conditions:
7
  - lastProbeTime: null
8
    lastTransitionTime: "2022-05-09T15:23:37Z"
9
    status: "True"
10
    type: Initialized                         #conditions状态2
11
  - lastProbeTime: null
12
    lastTransitionTime: "2022-05-09T15:23:51Z"
13
    status: "True"
14
    type: Ready                               #conditions状态4
15
  - lastProbeTime: null
16
    lastTransitionTime: "2022-05-09T15:23:51Z"
17
    status: "True"
18
    type: ContainersReady                     #conditions状态3
19
  - lastProbeTime: null
20
    lastTransitionTime: "2022-05-09T15:23:37Z"
21
    status: "True"
22
    type: PodScheduled                        #conditions状态1
23
  containerStatuses:
24
  - containerID: docker://9d56be349349b7581a4178b11895167b5be8c1c68ce1630f440389a1e8257a35
25
    image: docker.io/rancher/curl:latest
26
    imageID: docker-pullable://docker.io/rancher/curl@sha256:85aea1846e2e9b921629e9c3adf0c5aa63dbdf13aa84d4dc1b951982bf42d1a4
27
    lastState: {}
28
    name: main
29
    ready: true
30
    restartCount: 0
31
    started: true
32
    state:
33
      running:
34
        startedAt: "2022-05-09T15:23:50Z"
35
  hostIP: 172.16.11.161
36
  phase: Running
37
  podIP: 10.244.2.245
38
  podIPs:
39
  - ip: 10.244.2.245
40
  qosClass: BestEffort
41
  startTime: "2022-05-09T15:23:37Z"
42
[root@master-node ~]#
从上，我们可以从yaml中的Status字段里的conditions字段看到pod的4个conditions。

同样，我们也可以通过kubectl describe pod来获取conditions：kubectl describe pod pod_name|grep Conditions: -A 5

1
[root@master-node ~]# kubectl describe pod curl|grep Conditions: -A 5
2
Conditions:
3
  Type              Status
4
  Initialized       True 
5
  Ready             True 
6
  ContainersReady   True 
7
  PodScheduled      True 
8
[root@master-node ~]# kubectl describe pod my-job-jfhz9|grep Conditions: -A5
9
Conditions:
10
  Type              Status
11
  Initialized       True 
12
  Ready             False 
13
  ContainersReady   False 
14
  PodScheduled      True 
15
[root@master-node ~]# 
表示要过滤Conditions:字段，然后，-A5表示紧随其后的5行。-A 5之间也可以留空格，不影响结果。

比如：上述我们看到的类型为job的pod my-job-jfhz9它的conditions中，有多个是False的。为什么呢？我们同样，可以从kubectl get pods pod_name -oyaml里查看到类似下述的说明信息：

 conditions:
    - lastProbeTime: null
      lastTransitionTime: "2022-05-09T15:22:23Z"
      reason: PodCompleted
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2022-05-09T15:24:44Z"
      reason: PodCompleted
      status: "False"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2022-05-09T15:24:44Z"
      reason: PodCompleted
      status: "False"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2022-05-09T15:22:19Z"
      status: "True"
      type: PodScheduled
我们，从reason字段里，看到pod处于某个状态下的具体原因。原来，ContainersReady的状态为false的原因，是PodCompleted了。

其中的lastProbeTime表示的该conditions在什么时间被检查过，名字中虽然有probe，但是它跟probe没有关系；

lastTransitionTime表示该conditions在什么时间点儿发生的变化；