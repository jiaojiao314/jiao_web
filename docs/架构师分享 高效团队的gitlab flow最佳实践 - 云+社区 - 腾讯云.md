参考链接
[https://cloud.tencent.com/developer/article/1792828?from=article.detail.1689274](https://cloud.tencent.com/developer/article/1792828?from=article.detail.1689274)


当前 git 是大部分开发团队的首选版本管理工具，一个好的流程规范可以让大家有效地合作，像流水线一样有条不紊地进行团队协作。

下面我们先来分析，然后再看我们团队基于 gitlab flow 的最佳实践。

# 从git flow到gitlab flow
# git flow


先说 git flow，大概是这样的。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627628992106-2ee1a870-6302-4f72-add1-f76fa0b83d02.png#height=700&id=pRwTs&margin=%5Bobject%20Object%5D&name=image.png&originHeight=700&originWidth=528&originalType=binary&ratio=1&size=252882&status=done&style=none&width=528)

然后，我们老的 git 规范是参考 git flow 实现的。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627628995497-726a3ce3-5774-41af-99b6-616635575b44.png#height=616&id=vCug9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=616&originWidth=720&originalType=binary&ratio=1&size=117954&status=done&style=none&width=720)

综合考虑了开发、测试、新功能开发、临时需求、热修复，理想很丰满，现实很骨干，这一套运行起来实在是太复杂了。那么如何精简流程呢？

我们来看业界的做法，首先是 github flow。

Github flow 是 Git flow 的简化版，专门配合” 持续发布”。它是 Github.com 使用的工作流程。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629004795-945c6446-fe15-4dd9-a9b5-21832f28f7f0.png#height=467&id=tPj1f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=467&originWidth=380&originalType=binary&ratio=1&size=15758&status=done&style=none&width=380)

整个流程：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629012813-b2732e73-2922-4afe-bfe4-f178ca70ce14.png#height=138&id=O4fOy&margin=%5Bobject%20Object%5D&name=image.png&originHeight=138&originWidth=750&originalType=binary&ratio=1&size=22151&status=done&style=none&width=750)

- 第一步：根据需求，从 master 拉出新分支，不区分功能分支或补丁分支。
- 第二步：新分支开发完成后，或者需要讨论的时候，就向 master 发起一个 pull request（简称 PR）。
- 第三步：Pull Request 既是一个通知，让别人注意到你的请求，又是一种对话机制，大家一起评审和讨论你的代码。对话过程中，你还可以不断提交代码。
- 第四步：你的 Pull Request 被接受，合并进 master，重新部署后，原来你拉出来的那个分支就被删除。（先部署再合并也可。）

github flow 这种方式，要保证高质量，对于贡献者的素质要求很高，换句话说，如果代码贡献者素质不那么高，质量就无法得到保证。

> github flow 这一套对于库、框架、工具这样并非最终应用的产品来说，没问题，但是，如果如果一个产品是 “最终应用”，github flow 可能就不合适了。


# gitlab flow

Gitlab flow 是 Git flow 与 Github flow 的综合。它吸取了两者的优点，既有适应不同开发环境的弹性，又有单一主分支的简单和便利。它是 Gitlab.com 推荐的做法。

Gitlab flow 的最大原则叫做” 上游优先”（upsteam first），即只存在一个主分支 master，它是所有其他分支的” 上游”。只有上游分支采纳的代码变化，才能应用到其他分支。

对于” 持续发布” 的项目，它建议在 master 分支以外，再建立不同的环境分支。比如，” 开发环境” 的分支是 master，” 预发环境” 的分支是 pre-production，” 生产环境” 的分支是 production。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629025086-fc449294-8aa9-45e6-93e4-933ca120d3c3.png#height=618&id=r4AXC&margin=%5Bobject%20Object%5D&name=image.png&originHeight=618&originWidth=560&originalType=binary&ratio=1&size=32895&status=done&style=none&width=560)

只有紧急情况，才允许跳过上游，直接合并到下游分支。

对于” 版本发布” 的项目，建议的做法是每一个稳定版本，都要从 master 分支拉出一个分支，比如 2-3-stable、2-4-stable 等等。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629033138-82f7f507-2394-45ab-9b3e-762b2b39e6cd.png#height=719&id=u0KCy&margin=%5Bobject%20Object%5D&name=image.png&originHeight=719&originWidth=550&originalType=binary&ratio=1&size=33664&status=done&style=none&width=550)

gitlab flow 如何处理 hotfix？git flow 之所以这么复杂，一大半原因就是把 hotfix 考虑得太周全了。hotfix 的意思是，当代码部署到产品环境之后发现的问题，需要火速 fix。gitlab flow 可以基于后续分支，修改后上线。
# 团队git规范

综合上面的介绍，我们决定采用 gitlab flow，按照版本发布的模式实施，具体来说：

1. 新的迭代开始，所有开发人员从主干 master 拉个人分支开发特性, 分支命名规范 feature-name
2. 开发完成后，在迭代结束前，合入 master 分支
3. master 分支合并后，自动 cicd 到 dev 环境
4. 开发自测通过后，从 master 拉取要发布的分支，release-$version，将这个分支部署到测试环境进行测试
5. 测出的 bug，通过从 release-versio 拉出分支进行修复，修复完成后，再合入 release−versio
6. 正式发布版本，如果上线后，又有 bug，根据 5 的方式处理
7. 等发布版本稳定后，将 release-$versio 反合入主干
# 最佳实践
# 开发feature功能

新建分支，比如 feat-test

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629043671-eaa9af4a-88e7-410d-a38e-027922420bcb.png#height=244&id=KzF0J&margin=%5Bobject%20Object%5D&name=image.png&originHeight=244&originWidth=471&originalType=binary&ratio=1&size=13795&status=done&style=none&width=471)


# 提交MR
开发代码，增加新功能，提交：

```
@GetMapping(path \= "/test", produces \= "application/json")
	@ResponseBody
	public Map<String, Object\> test() {
		return singletonMap("test", "test");
	}
```

```
git commit \-m "feat: add test code"
git push origin feat\-test
```

提交代码后，可以提交 mr 到 master，申请合并代码

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629052090-e4b32b9f-5bc3-4e13-af03-4761c30fc982.png#height=765&id=S56Rv&margin=%5Bobject%20Object%5D&name=image.png&originHeight=765&originWidth=1080&originalType=binary&ratio=1&size=90829&status=done&style=none&width=1080)
Note：

- 这里可以增加自动代码审查,

# 合并代码
研发组长，打开 mr，review 代码，可以添加建议：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629066986-9f75072b-2914-452e-88df-5661ac0cf2c2.png#height=587&id=bXG5o&margin=%5Bobject%20Object%5D&name=image.png&originHeight=587&originWidth=1080&originalType=binary&ratio=1&size=125483&status=done&style=none&width=1080)

开发同学根据建议修复代码，或者线下修改后 commit 代码。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629073482-2ac74ab0-31e7-477b-b26b-e8b09e5cd717.png#height=473&id=cSznN&margin=%5Bobject%20Object%5D&name=image.png&originHeight=473&originWidth=1014&originalType=binary&ratio=1&size=52650&status=done&style=none&width=1014)

研发组长确认没有问题后，可以合并到 master。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/671316/1627629079332-82f6e411-c278-4c1c-8d4e-50868be7bade.png#height=440&id=hyMOB&margin=%5Bobject%20Object%5D&name=image.png&originHeight=440&originWidth=958&originalType=binary&ratio=1&size=31256&status=done&style=none&width=958)

合并完成，可以删除 feat 分支。

新功能开发好，可以进行提测。

# 发布版本
# 语义化版本号

版本格式：主版本号. 次版本号. 修订号，版本号递增规则如下：

主版本号：当你做了不兼容的 API 修改， 次版本号：当你做了向下兼容的功能性新增， 修订号：当你做了向下兼容的问题修正。先行版本号及版本编译元数据可以加到 “主版本号. 次版本号. 修订号” 的后面，作为延伸。


主版本号为 0，代表还未发布正式版本。
# 测试发布
master 分支，自动部署到开发环境（dev）

功能开发完成，并自测通过后，代码合并到待发布版本，

分支规则：

```
release-version
```

版本规则

```
主版本号.次版本号
```

构建时，自动增加修订号：

```
主版本号.次版本号.修订号
```

从最新的 master 新拉一个分支 release-$version，比如 release-0.1

```
git checkout -b release-0.1
```

release-version 会自动构建，版本号为 version.$buildNumber

设定 release-$version 分支为保护分支，不允许直接推送，只能通过 merge 不允许直接提交代码，接受 MR。

# bug修复
需要修改 bug 时，从 release-version 新拉分支，修改完成后再合并到 release−version 分支.

- Q: 从 release-$version 拉的分支，如何测试？
- A: 这个节点定义为 bug 修复节点，建议开发同学优先本地测试验证，严重通过再合并到 release 分支。
- Q: release-$version 太多怎么办？
- A: 可以保留最近的 10 个版本。历史的打 tag 后，删除分支。
[https://cloud.tencent.com/developer/article/1792828?from=article.detail.1689274](https://cloud.tencent.com/developer/article/1792828?from=article.detail.1689274)
