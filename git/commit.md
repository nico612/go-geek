git commit message 规范基本使用[Angular规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#commits)

### 规范内容
一个规范的 Git Commit Message 通常包含一下内容：

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- 类型（type）：用于说明 commit 的类别。
- 范围（scope）：用于说明 commit 影响的范围，比如更改了哪个文件、哪个函数等。
- 主题（subject）：简短描述本次 commit 的内容，不超过 50 个字符。
- 描述（body）：对本次 commit 进行详细的描述，可以分成多行。
- 备注（footer）：关联 issue 问题链接编号等；

| Type     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| feat     | 新增特性（feature）                                          |
| fix      | 修复 Bug（bug fix）                                          |
| docs     | 修改文档（documentation）                                    |
| style    | 代码格式修改(white-space, formatting, missing semi colons, etc) |
| refactor | 代码重构(refactor)                                           |
| perf     | 改善性能(A code change that improves performance)            |
| test     | 测试(when adding missing tests)                              |
| build    | 变更项目构建或外部依赖（例如 scopes: webpack、gulp、npm 等） |
| ci       | 更改持续集成软件的配置文件和 package 中的 scripts 命令，例如 scopes: Travis, Circle 等 |
| chore    | 变更构建流程或辅助工具(比如更改测试环境)                     |
| revert   | 代码回退                                                     |

其中，类型和主题是必须的，其他选填；示例：

```
feat(views): Add ErrorPage component 

Add a global error page.

```

```shell
feat(登录页): 增加 Google 登录方式

在登录页中增加了使用 Google 账号登录的功能。
```

### commitizen 插件

官方地址：https://github.com/commitizen/cz-cli

为了更加方便地遵守 `Git Commit `规范，我们可以使用 `commitizen` 插件。`commitizen` 是一个专门用于规范化 `Git Commit` 记录的工具，它可以在提交代码时帮助我们生成符合规范的 `Git Commit` 记录。

### 全局安装

```
npm install -g commitizen
```

初始化：选择 `cz-conventional-changelog` 这个预设初始化，它与我们前面介绍的 Git Commit 规范一致。

```
commitizen init cz-conventional-changelog --save-dev --save-exact
```

使用：用 git cz 来代替 git commit 命令来提交代码。

```shell
git cz
```

#### 项目安装

安装：项目根目录下，安装命令。

```shell
npm install commitizen -D
```

初始化：利用 `npx` 执行 `commitizen` 命令中使用 `cz-conventional-changelog` 预设，如果你的npm版本在5.2以上，那么你可以使用npx来初始化：；

```shell
npx commitizen init cz-conventional-changelog --save-dev --save-exact
```

使用：用 npx cz 来代替 git commit 命令来提交代码。

```shell
npx cz
```

扩展：可在 package.json 的 scripts 中添加命令来 npm run 执行；

```shell
{
  "scripts": {
    "...": "...",
    "commit": "git add . && cz"
  }
}

```

提交代码；

```
npm run commit
# 等价于
git add .
npx cz
```

#### 使用指南

执行 cz 命令提交代码时，commitizen 会在终端中发起如下会话，根据提示选择合适的type，填写scope、subject等；

```shell
# 选择type，本次 commit 的类型；
Select the type of change that you're committing:

# 输入 scope 影响范围（回车可跳过）；
What is the scope of this change (e.g. component or file name): (press enter to 
skip) 

# 输入 subject 主题，简短描述 commit 内容；
Write a short, imperative tense description of the change (max 94 chars):

# 输入 body 描述，详细描述 commit 内容（回车可跳过）；
Provide a longer description of the change: (press enter to skip)

# 本次 commit 是否是一次重大的更改（y/N）；
Are there any breaking changes? (y/N) 

# 本次 commit 是否影响哪个issues（y/N）；
Does this change affect any open issues? (y/N)

```











