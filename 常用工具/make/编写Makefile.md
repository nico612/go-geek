Makefile 集成常用功能

```shell
Usage:
 make [target]

Targets:
build                  Build source code for host platform
buildmultiarch         Build source code for multiple platforms
image                  Build docker images for host arch
push                   Build docker images for host arch and push images to registry.
deploy                 Deploy updated components to development env.
install                Install system with all its components.
clean                  Remove all files that are created by building.
gen                    gen: Generate all necessary files, such as error code files.
ca                     ca: Generate CA files for all iam components.
lint                   Check syntax and styling of og sources.
test                   Run unit test.
cover                  Run unit test and get test coverage.
format                 format: Gofmt (reformat) package sources (exclude vendor dir if existed).
verify-copyright       verify-copyright: Verify the boilerplate headers for all files.
add-copyright          add-copyright: Ensures source code files have copyright license headers.
swagger                swagger: Generate swagger document.
serve-swagger          serve-swagger: Serve swagger spec and docs.
check-updates          check-updates: Check outdated dependencies of the go projects.
help                   show help
```



常用变量设置

```makefile

SHELL := /bin/bash

# include the common make file
COMMON_SELF_DIR := $(dir $(lastword $(MAKEFILE_LIST)))


# 项目根目录
ifeq ($(origin ROOT_DIR),undefined)
ROOT_DIR := $(abspath $(shell cd $(COMMON_SELF_DIR)/../.. && pwd -P))
endif

# 创建临时输出目录
ifeq ($(origin OUTPUT_DIR),undefined)
OUTPUT_DIR := $(ROOT_DIR)/_output
$(shell mkdir -p $(OUTPUT_DIR))
endif
ifeq ($(origin TOOLS_DIR),undefined)
TOOLS_DIR := $(OUTPUT_DIR)/tools
$(shell mkdir -p $(TOOLS_DIR))
endif
ifeq ($(origin TMP_DIR),undefined)
TMP_DIR := $(OUTPUT_DIR)/tmp
$(shell mkdir -p $(TMP_DIR))
endif


# set the version number. you should not need to do this
# for the majority of scenarios.
ifeq ($(origin VERSION), undefined)
VERSION := $(shell git describe --tags --always --match='v*')
endif

# Check if the tree is dirty. default to dirty

GIT_TREE_STATE:="dirty"
ifeq (, $(shell git status --porcelain 2>/dev/null))
	GIT_TREE_STATE="clean"
endif
GIT_COMMIT:=$(shell git rev-parse HEAD)


# The OS must be linux when building docker images
# PLATFORMS ?= linux_amd64 linux_arm64
# The OS can be linux/windows/darwin when building binaries
 PLATFORMS ?= darwin_amd64 darwin_arm64 windows_amd64 linux_amd64 linux_arm64

# 设置指定平台
ifeq ($(origin PLATFORM), undefined)
	ifeq ($(origin GOOS), undefined)
		GOOS := $(shell go env GOOS)
	endif
	ifeq ($(origin GOARCH), undefined)
		GOARCH := $(shell go env GOARCH)
	endif
	PLATFORM := $(GOOS)_$(GOARCH)
	# Use linux as the default OS when building images
	IMAGE_PLAT := linux_$(GOARCH)
else
	GOOS := $(word 1, $(subst _, ,$(PLATFORM)))
	GOARCH := $(word 2, $(subst _, ,$(PLATFORM)))
	IMAGE_PLAT := $(PLATFORM)
endif



```

