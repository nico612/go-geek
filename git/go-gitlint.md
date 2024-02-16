#  go-gitlint 使用指南



## 安装：

```shell
go get github.com/marmotedu/go-gitlint/cmd/go-gitlint
```

## 配置

### githook: commit-msg配置

```shell

# commit-msg use go-gitlint tool, install go-gitlint via `go get github.com/llorllale/go-gitlint/cmd/go-gitlint`
go-gitlint --msg-file="$1"
```

### .gitlint配置

```bash
--subject-regex=^(revert: )?(feat|fix|perf|style|refactor|test|ci|docs|chore)(\(.+\))?: [^A-Z]*[^.]$
--subject-maxlen=72
--body-regex=.*
--body-maxlen=72
```

## 运行

```bash
$ cd ${PROJECT_ROOT}
$ go-gitlint
```