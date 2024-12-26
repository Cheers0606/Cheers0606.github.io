---
title: hello world
date: 2018-07-17 12:33:59
updated: 2021-01-08 12:33:59
tags: ["hexo demo"] # 好像没用
overdue: true #这一行
categories: 测试 # 目录
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

### 流程图
```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```


### class disagram
```mermaid
classDiagram
    Class01 <|-- AveryLongClass : Cool
    Class03 *-- Class04
    Class05 o-- Class06
    Class07 .. Class08
    Class09 --> C2 : Where am i?
    Class09 --* C3
    Class09 --|> Class07
    Class07 : equals()
    Class07 : Object[] elementData
    Class01 : size()
    Class01 : int chimp
    Class01 : int gorilla
    Class08 <--> C2: Cool label
```

### sequence
```mermaid
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
```
### gantt

```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title Adding GANTT diagram to mermaid
    
    section A section
    Completed task            :done,    des1, 2014-01-06,2014-01-08
    Active task               :active,  des2, 2014-01-09, 3d
    Future task               :         des3, after des2, 5d
    Future task2               :         des4, after des3, 5d
```

### mermaid diagrams
```mermaid
graph TD;
    A[test]
    B[hello]
    D-->G
    A-->B
    A-->C
    B-->D
    C-->D
	subgraph test
	E-->G
	end
```
### graph TD

```mermaid
graph TD;
	runJob-->ostart
	ostart-->jstart
	processRuleUpdatedLabel--是-->cleanStart
	subgraph all
	start[开始]-->edit[修改标签]
	edit--执行标签任务-->redis[提交到redis的allJob队列]
	redis--JobAcceptor#parseMessage-->runJob[执行标签 调用海纳]
	end

	subgraph ocean[海纳任务]
	ostart[海纳task]-->oend[海纳任务执行完成]
	end

	subgraph JobTracker
	jstart[从redis的scheduler:running获取运行中的任务]-->isJobEnd{任务是否到达终态}
	isJobEnd--是-->updateLabelCoverage[更新标签覆盖数]-->processRuleUpdatedLabel{判断规则是否改过}
	updateLabelCoverage-->processOneTimeLabelStatus[一次性标签需要修改oneTimeStatus状态位为已执行]-->removeFromRedisSet[从redis的running中删除]-->oend
	isJobEnd--否-->jstart
	end

	subgraph ofCleanOldRuleJob[清理修改过规则后的过期标签]
	cleanStart-->cleanEnd
	end
```