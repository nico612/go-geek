

RSA 加密演算法是一种非对称加密演算法[[1\]](https://www.codeover.cn/go_rsa/#fn:1)，在一些项目中经常使用，是目前使用最广的数据安全加密算法之一。在 golang 中， RSA 的加密、解密、签名与验签主要使用 `crypto/x509` 和 `crypto/rsa` 两个包中的方法。

## 公私钥的生成

### 使用**OpenSSL**s生成

**OpenSSL**：这是一个功能强大的开源工具包，支持各种加密算法，包括RSA。通过命令行或API，可以生成RSA密钥对。

openSSL工具使用方法见: [OpenSSL](/Users/admin/work/sources/go/常用工具/OpenSSL.md) 

## 加密与解密

```go

var privateKeyBytes []byte
var publicKeyBytes []byte

var privateKey *rsa.PrivateKey

func init() {
	var err error
  
  // 读取私钥文件
	privateKeyBytes, err = utils.ReadFile("xxxx.key")
	if err != nil {
		panic(err)
	}
  
  // 读取公钥文件
	publicKeyBytes, err = utils.ReadFile("xxxx.pub")
	if err != nil {
		panic(err)
	}

  // 解析私钥
	block, _ := pem.Decode(privateKeyBytes)
	b := block.Bytes
	key, err := x509.ParsePKCS8PrivateKey(b)
	if err != nil {
		panic(err)
	}
	if err != nil {
		panic(err)
	}
	
	privateKey = key.(*rsa.PrivateKey)
}

func Pubkey() string {
	return string(publicKeyBytes)
}

func Decrypt(value []byte) (string, error) {
	decryptedBytes, err := rsa.DecryptPKCS1v15(rand.Reader, privateKey, value)
	if err != nil {
		return "", errors.WithStack(err)
	}

	return string(decryptedBytes), nil
}

// Encrypt rsa 公钥加密，返回编码为 base64 结果
func Encrypt(value []byte) (string, error) {
	encryptedData, err := rsa.EncryptPKCS1v15(rand.Reader, &privateKey.PublicKey, value)
	if err != nil {
		return "", err
	}
	return base64.StdEncoding.EncodeToString(encryptedData), nil
}

// DecodePassword 解密， pwd 为 rsa公钥加密后 再 base64编码的
func DecodePassword(pwd string) (string, error) {
	if pwd == "" {
		return "", nil
	}
	b, err := base64.StdEncoding.DecodeString(pwd)
	if err != nil {
		return "", bizerr.InvalidArgument().Msg("decode password failed")
	}

	plaintext, err := Decrypt(b)
	if err != nil {
		return "", bizerr.InvalidArgument().Msg("decrypt password")
	}
	return plaintext, err
}

```



## 签名与验签

签名和验签是常见的加密通信中的重要步骤，用于确保数据的完整性和来源可信性。在加密通信中，签名用于对数据进行加密，而验签则用于验证签名的有效性。

使用 RSA 私钥对数据进行签名，然后使用公钥进行验签。以下是一个示例代码，展示了如何使用 RSA 进行签名和验签：

```go
package main

import (
	"crypto"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"fmt"
)

func main() {
	// 生成密钥对
	privateKey, err := rsa.GenerateKey(rand.Reader, 2048)
	if err != nil {
		fmt.Println("Error generating RSA key:", err)
		return
	}

	// 待签名的数据
	message := []byte("Hello, this is a message to be signed.")

	// 使用 SHA256 算法对数据进行签名
	hashed := sha256.Sum256(message)
	signature, err := rsa.SignPKCS1v15(rand.Reader, privateKey, crypto.SHA256, hashed[:])
	if err != nil {
		fmt.Println("Error signing:", err)
		return
	}
	fmt.Printf("Signature: %x\n", signature)

	// 模拟传输数据及签名
	receivedMessage := message // 假设收到传输的数据
	receivedSignature := signature // 假设收到传输的签名

	// 验证签名
	hashedReceived := sha256.Sum256(receivedMessage)
	err = rsa.VerifyPKCS1v15(&privateKey.PublicKey, crypto.SHA256, hashedReceived[:], receivedSignature)
	if err != nil {
		fmt.Println("Signature verification failed:", err)
		return
	}
	fmt.Println("Signature verified successfully!")
}

```



















