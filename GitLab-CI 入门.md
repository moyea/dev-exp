## CI/CD development documentation

https://docs.gitlab.com/ee/development/cicd/

![ci](https://docs.gitlab.com/ee/development/cicd/img/ci_architecture.png)

当触发Pipeline Triggers中的事件时会调用`CreatePipelineService`来接受输入事件，尝试创建一个Pipeline。

`CreatePipelineService`依赖`YAML Processer`组件，接收YAML blob作为输入，并返回Pipeline的抽象数据结构(包括stages和jobs)。`YAML Processer`也在处理YAML时验证它的结构，并返回任何语法或语义错误。`CreatePipelineService`接收`YAML Processer`返回的抽象数据结构，将其转换为持久化模型(Pipelines, Stages, Jobs等)。在此之后，就是处理pipeline了。也就是按stage中定义的jobs顺序来执行相应的job，直到所有预期的job被执行或者故障中断了pipeline的执行。

`ProcessPipelineService`用于处理pipeline，它负责将管道的所有job转移到完成状态。创建pipeline时，其所有job最初都处于`created`状态。此服务根据pipeline结构查看在`craated`状态中哪些job有资格被处理。然后它会将它们移动到`pending`状态，这意味着它们现在可以被一个Runner拾起。job执行完成后，它可以成功完成，也可以失败。`pipeline`中job的每个状态转换都会再次触发此服务，该服务将查看下一个job是否转换到完成状态。同时，ProcessPipelineService 更新jobs、stages和整个pipeline的状态。

同时也有一组链接到Gitlab instance的`Runners`列表，包括Shared Runner, Group Runner, Specific Runner。Runners和Rails服务器之间通过一组API端点进行通信，也就是`Runner API Gateway`。

我们可以注册、删除和验证 Runners，这也会导致对数据库的读/写查询。当一个Runner连接上后，它会不断地要求执行下一个job。这将调用 `RegisterJobService`，它将挑选下一个job并将其分配给 Runner。此时，job将转换为`running`状态，由于状态更改，这将再次触发 ProcessPipelineService。

在执行job时，Runner 会将日志以及需要存储的任何可能的`Artifacts`发送回服务器。此外，为了运行，job可能依赖于前面job中的`Artifacts`。在这种情况下，Runner 将使用一个专用的 API 端点下载它们。`Artifacts`存储在对象存储中，而元数据存储在数据库中。

Job状态转换并非完全自动化。用户可以运行手动job、取消pipeline、重试特定失败的job或整个pipeline。任何导致job更改状态的操作都将触发 ProcessPipelineService，因为它负责跟踪整个pipeline的状态。

一种特殊类型的job是`bridge job`，它在转换到`pending`状态时在服务器端执行。该工作负责创建下游pipeline，如多项目pipeline或子pipeline。每次触发下游pipeline时，工作流循环都从 CreatePipelineService 重新启动。

## Runner

GitLab Runner是一个开源项目，用于执行你的任务并将结果返回给GitLab. 与GitLab CI/CD 一起使用，包含开源持续集成服务来协调你的任务。

[GitLab Runner](https://docs.gitlab.com/runner/)

#### 安装Runner

1. Install the GitLab Runner.

   ```bash
   brew install gitlab-runner
   ```

2. Install the Runner as a service and start it.

   ```bash
   brew services start gitlab-runner
   ```

#### 配置Runner

[Register GitLab Runner](https://docs.gitlab.com/runner/register/index.html)

[Configuring GitLab Runners](https://docs.gitlab.com/ee/ci/runners)

https://docs.gitlab.com/ee/ci/caching/index.html#clearing-the-cache

#### 构建目录

GitLab Runner 会将repository目录克隆并存储基本目录下，也就是*Builds Directory* 下. 基本路径的默认位置取决于具体执行的Runner. For:

 GitLab Runner 会 *Builds Directory* 来运行它将要运行的所有Job, 但是有一个特定的嵌套规则 `{builds_dir}/$RUNNER_TOKEN_KEY/$CONCURRENT_ID/$NAMESPACE/$PROJECT_NAME`. eg: `/builds/2mn-ncv-/0/user/playground`

#### Job调度

管道被创建时，它所有stages的jobs也会同时创建，初始状态为`create`。该状态下的job可以被用户看到。

具有`create`状态的jobs还不会被Runner看到。为了能够将job分配给一个 Runner，job必须首先转换到`pending`状态，如果:

1. 这个工作是在管道的第一阶段创建的
2. 这项工作需要手动启动，现在已经启动了
3. 前一阶段的所有工作均已顺利完成。在这种情况下，我们将所有的工作从下一个阶段转移到`pending`
4. jobs中`needs:`标记的所有依赖job已完成

当连接到 Runner 时，它请求通过不断轮询服务器来运行下一个`pending`的job。



## 配置GitLab CI/CD

官方文档: https://docs.gitlab.com/ee/ci/yaml/README.html

中文翻译: https://segmentfault.com/a/1190000010442764

GitLab CI/CD  pipeline是在每个项目中使用名为`.gitlab-ci. yml`的 YAML 文件配置的。该文件定义了pipeline的结构和顺序，并确定了:

- 使用GitLab Runner来执行什么
- 遇到特定条件时该做出什么样的决策。例如：当一个流程成功或失败时

在`.gitlab-ci.yml`中有一些基本概念

### Pipeline

一个PIpeline相当于一次构建任务，里面可以包含多个流程，比如安装依赖，编译代码，部署到生产环境等。任何Pipeline Trigger 事件(如commit, MR等)都会触发Pipeline的创建。



### Stages

stages表示构建阶段，也就是上面的流程。一个Pipeline中可以定义多个stages，每个stage可以执行不同的job。Stages有以下几个特点:

- 所有Stages会按顺序执行，即一个Stage完成后才会开始下一个Stage
- 只有当所有Stages完成后，该构建任务(Pipeline)才会成功
- 如果任何一个Stage失败, 那么后面的Stages不会执行, 该构建任务(Pipeline)失败

### Jobs

Pipeline配置从 Jobs 开始，Jobs 是`.gitlab-ci. yml`文件中最基本的元素。jobs包括：

- 定义了约束，说明在什么条件下应该执行这些约束
- 具有任意名称的顶级元素，并且必须至少包含[`script`](https://docs.gitlab.com/ee/ci/yaml/README.html#script)子句
- 不限制可以定义的数量
- 一个job若不指定`stage:`(或`type:`已废弃)，则默认为stage为 test

例如:

```yaml
job1:
	script: "echo job1"

job2:
	script: "echo job2"
```

上面的例子是两个独立job的最简单的 CI/CD 配置，其中每个job执行不同的命令。当然，命令可以直接执行代码(。/configure; make; make install)或在存储库中运行脚本(test.sh)。

jobs会被runner获取, 并在runner的环境中执行。同时每个job是独立运行的。



#### Pipeline, Stages和Jobs之间的关系

```
+-----------------------------------------------------------+
|                                                           |
|  Pipeline                                                 |
|                                                           |
|  +-----------+     +------------+      +------------+     |
|  |  Stage 1  |     |   Stage 2  |      |   Stage 3  |     |
|  |           |     |            |      |            |     |
|  |  +------+ |     |  +------+  |      |  +------+  |     |
|  |  | Job1 | |     |  | Job3 |  |      |  | Job5 |  |     |
|  |  +------+ | --> |  +------+  | ---> |  +------+  | ... |
|  |  | Job2 | |     |  | Job4 |  |      |  | Job6 |  |     |
|  |  +------+ |     |  +------+  |      |  +------+  |     |
|  |    ...    |     |     ...    |      |     ...    |     |
|  +-----------+     +------------+      +------------+     |
|                                                           |
+-----------------------------------------------------------+
```



### 验证`.gitlab-ci.yml`

每个 GitLab CI/CD 实例都有一个名为 Lint 的嵌入式调试工具，它可以验证`.gitlab-ci.yml` 文件. 可以在项目名称空间的页面 ci/lint 下找到 Lint。例如：https://git.llsapp.com/sprout/game/pharics/-/ci/lint 

## Cache vs artifacts

[Cache vs artifacts](https://docs.gitlab.com/ee/ci/caching/index.html#cache-vs-artifacts)

> 在job中如果同时使用cache和artifacts存储相同的path, 由于cache先于artifacts存储，所以content可能会被更改

不要使用cache传递stages的artifacts，cache的设计目的在于传递编译项目运行时所需的依赖

#### cache和artifacts的用途

- Cache : 用于存储项目的依赖

  Cache用于加速一个Pipeline的后续job执行，通过存储下载的depenencies，后续的job不需要再次从网络上获取(eg. npm packages等)，然而cache也可以配置为传递一些stages中build的结果，这个实际上应该是`artifacts`应该做的事。

- Artifacts: 用与传递stages与stages间的结果

  Artifacts是被job创建出来的一些文件并且被保存并上传的，可以同一个pipeline后续stages获取到。也就是说你不能在stage-1的job-A中创建一个artifacts，然后再stage-1的job-B中使用这个artifacts。同时这部分数据在不同的pipeline中也无法使用，但是可以再WebUI界面下载它。

#### cache与artifacts对比

**Caches:**

- 如果没有(使用`cache:`)在全局或每个job中声明会被禁用
- 在`.gitlab-ci.yml`全局启用的在每个job中都可用
- (非全局)创建的可以被后续的pipeline相同的job使用
- 存储在Runner安装位置并且在配置runner cache server的情况下会上传到cache server
- 如果在每个job里创建，可用于：
  - 后续pipeline的相同job
  - 同一个pipeline后续有相同dependencies的job

**Artifacts:**

- 如果job中未(使用`artifacts:`)定义则会被禁用
- 只能被每个job中使用, 无法全局使用
- 随着一个pipeline被创建的只能用于当前pipeline后续的jobs
- 总是会被上传到GitLab(被称为coordinator)
- 可以有一个过期时间来控制磁盘的使用(默认为30天)

> artifacts和caches中定义的paths都依赖于项目目录，无法连接到外部文件



