## GitHub Actions 入门

一般来说，持续集成由很多操作组成，比如抓取代码、运行测试、登录远程服务器、发布到第三方服务等。GitHub 把这些操作就称为actions。

很多操作在不同的项目中是类似的，完全可以共享。GitHub 注意到这一点，想出了一个很妙的点子，允许开发者把每个操作写成独立的脚本文件，存放到代码仓库，使得其他开发者可以使用。

GitHub Action 帮助我们在存储代码的同一位置自动执行软件开发的工作流程，并协作处理 Pull Requests 和 issues 。我们可以写入个别任务(即action)，并结合它们创建一个自定义的 workflow 。workflows 是我们可以在仓库中创建的自定义自动化流程，用于在 GitHub 上构建、测试、打包、发布、或部署任何代码项目。

通过 GitHub Actions 可直接在仓库中构建端到端持续集成(CI)和持续部署(CD)功能。

Workflows 在 GitHub 托管的计算机(即 Runner )上的 Linux、macOS、Windows 和容器上运行。当然，也可以托管自己的 Runner，让 workflows 在自己拥有或管理的计算机上运行。[关于自托管 Runner ](https://docs.github.com/cn/actions/hosting-your-own-runners/about-self-hosted-runners)

我们可以使用定义在我们 repo 中的 actions、开源的 actions、或者已经发布的 Docker 镜像来创建Workflows。在 forked repo 中，Workflows 默认不执行。

### 基本概念

| 术语     | 解释                                                         |
| -------- | ------------------------------------------------------------ |
| Action   | Workflow 中最小可移植单元，可以组合在一起创建workflow，在 workflow 中表现为 steps中的一个step |
| Artifact | 测试或构建过程中创建的一些文件。例如，artifacts 可能包括二进制或软件包文件，测试结果，屏幕截图或日志文件。artifacts 与创建它们的 workflow 运行相关联，并且可以由其他作业使用或部署。 |
| Workflow | 一次持续集成的运行过程                                       |
| Job      | 一个workflow 由一个或多个job构成                             |
| Step     | 每个Job由一个或多个step构成，每个step按顺序执行              |



## Workflow文件

Github Actions 的配置文件叫做workflow文件，存放在仓库的 `.github/workflows` 目录

workflow 文件采用 YAML 格式，文件名任意，后缀为 `.yml` 。一个 repo 可以有多个 workflow 文件。GitHub 会找到 `.github/workflows` 目录中的所有 `.yml` 文件，当特定的事件触发时，GitHub 会自动执行相应的文件。

workflow 配置文件介绍:

main.yml

```yaml
name: GitHub Actions Demo # 可选workflow 的名称。默认为当前 workflow 的文件名

# 指定该 workflow 的触发条件，通常是某些事件。以下就指定了 push 事件触发该 workflow
# 完整的事件列表 https://docs.github.com/en/actions/reference/events-that-trigger-workflows
on: push

# 可选的一个 map 集合，在当前 workflow 所有 job 和 steps 中可用的环境变量
#env:
#	 k: v

# workflow主体文件，表示执行一项或多项job
jobs:
  # job_id
  job1: 
    # 可选，在GitHub展示的job名
  	name: job-1
  	# 必须，指定运行该job所需的虚拟机环境
  	runs-on: ubuntu-latest
  	# 可选，指定该job的依赖关系，也就是指定job的运行顺序
  	#needs: <other_job_id>
  	# 指定每个job的执行步骤
  	steps:
  		# step-1 使用actions/checkout@v2 获取仓库代码
  		- uses: actions/checkout@v2
  		# step-2，执行一个单行shell脚本
  		# 指定GitHub展示的step名称
  		- name: step-2
  		  # 指定step-2的环境变量
  		  #env: 
  		  #	 k: v
  		  # 指定运行的脚本
  			run: echo Hello GitHub Action!
  		# step-3, 执行一个多行脚本
  		- name: step-3
  			run: |
  				echo step-3 multi-line script line-1
  				echo step-3 multi-line script line-2
  		# step-4, 执行当前仓库中自定义的my-action脚本
  		- name: step-4
  			uses: ./.github/actions/my-action
  			# 指定脚本所需的参数
  			with:
  			  who-to-greet: 'World'
  #job2
	job2:
		name: job-2
		runs-on: ubuntu-latest
		needs: job1
		steps:
			- run: echo job-2
  	
```

 