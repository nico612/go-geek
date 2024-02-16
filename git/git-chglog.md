# git-chglog 使用指南

使用git-chglog需要配置：

1. CHANGELOG模板
2. git-chglog配置

## 安装

```bash
go install github.com/git-chglog/git-chglog/cmd/git-chglog@latest
```

## 使用

```bash
$ git-chglog --init
```

选项：

- What is the URL of your repository?: https://github.com/nico612/im-go-web
- What is your favorite style?: github
- Choose the format of your favorite commit message: (): -- feat(core): Add new feature
- What is your favorite template style?: standard
- Do you include Merge Commit in CHANGELOG?: n
- Do you include Revert Commit in CHANGELOG?: y
- In which directory do you output configuration files and templates?: .chglog

运行命令 `-o 指定输出文件`：

```bash
$ git-chglog -o CHANGELOG/CHANGELOG-0.1.md
```

**其它使用方法：**

```bash
$ git-chglog

  If <tag query> is not specified, it corresponds to all tags.
  This is the simplest example.

$ git-chglog 1.0.0..2.0.0

  The above is a command to generate CHANGELOG including commit of 1.0.0 to 2.0.0.

$ git-chglog 1.0.0

  The above is a command to generate CHANGELOG including commit of only 1.0.0.

$ git-chglog $(git describe --tags $(git rev-list --tags --max-count=1))

  The above is a command to generate CHANGELOG with the commit included in the latest tag.

$ git-chglog --output CHANGELOG.md

  The above is a command to output to CHANGELOG.md instead of standard output.

$ git-chglog --config custom/dir/config.yml

  The above is a command that uses a configuration file placed other than ".chglog/config.yml".
```

## 配置

.chglog/CHANGELOG.tpl.md

```shell
{{ range .Versions }}
<a name="{{ .Tag.Name }}"></a>
## {{ if .Tag.Previous }}[{{ .Tag.Name }}]({{ $.Info.RepositoryURL }}/compare/{{ .Tag.Previous.Name }}...{{ .Tag.Name }}){{ else }}{{ .Tag.Name }}{{ end }} ({{ datetime "2006-01-02" .Tag.Date }})

{{ range .CommitGroups -}}
### {{ .Title }}

{{ range .Commits -}}
* {{ if .Scope }}**{{ .Scope }}:** {{ end }}{{ .Subject }}
  {{ end }}
  {{ end -}}

{{- if .RevertCommits -}}
### Reverts

{{ range .RevertCommits -}}
* {{ .Revert.Header }}
  {{ end }}
  {{ end -}}

{{- if .NoteGroups -}}
{{ range .NoteGroups -}}
### {{ .Title }}

{{ range .Notes }}
{{ .Body }}
{{ end }}
{{ end -}}
{{ end -}}
{{ end -}}
```

config.ymal

```yaml
style: github
template: CHANGELOG.tpl.md
info:
  title: CHANGELOG
  repository_url: https://github.com/nico612/im-go-web
options:
  commits:
     filters:
       Type:
         - feat
         - fix
         - perf
         - refactor
  commit_groups:
     title_maps:
       feat: Features
       fix: Bug Fixes
       perf: Performance Improvements
       refactor: Code Refactoring
       docs: add documentation
  header:
    pattern: "^(\\w*)(?:\\(([\\w\\$\\.\\-\\*\\s]*)\\))?\\:\\s(.*)$"
    pattern_maps:
      - Type
      - Scope
      - Subject
  notes:
    keywords:
      - BREAKING CHANGE
```

当然也可以直接使用其配置