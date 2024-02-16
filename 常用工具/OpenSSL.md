[TOC]

OpenSSL 是一个开源的工具包，提供了多种加密、解密和安全通信的功能。它可以在命令行下执行各种加密操作，也可以通过其提供的库在不同编程语言中使用。以下是一些常见的 OpenSSL 使用方法和命令

`openssl` 命令使用方式可参考：[Openssl子命令 genrsa, rsa, req, x509命令详解](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Ft990423909%2Farticle%2Fdetails%2F120837032)。

## RSA 密钥生成

### 生成RSA公私钥

```shell
# 生成私钥, 默认 2048位
openssl genrsa -out parameter.key 2048 

# 生成公钥
openssl rsa  -in parameter.key  -pubout -out parameter.pub
```

将命令集成到Makefile中方便操作

```makefile

# include the common make file
MAKEFILE_SELF_DIR := $(dir $(lastword $(MAKEFILE_LIST)))

# 根路径
ifeq ($(origin ROOT_DIR),undefined)
ROOT_DIR := $(abspath $(shell cd $(MAKEFILE_SELF_DIR) && pwd -P))
endif

.PHONY: genRSA
genRSA:
	@echo "========> Generate RSA private key and public key files"
	@openssl genrsa -out $(ROOT_DIR)/certs/parameter.key 2048
	@openssl rsa -in $(ROOT_DIR)/certs/parameter.key -pubout -out $(ROOT_DIR)/certs/parameter.pub

```

## 椭圆曲线公私钥生成

这里生成一个用于jwt签名用的公私钥

```shell
.PHONY: genAuthKey
genAuthKey:
	@echo "========> Generate ECC private key and public key files"
	@openssl ecparam -name prime256v1 -genkey -noout -out $(ROOT_DIR)/certs/auth.key
	@openssl ec -in $(ROOT_DIR)/certs/auth.key -pubout -out $(ROOT_DIR)/certs/auth.pub

```

