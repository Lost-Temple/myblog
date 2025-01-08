---
title: Fabric源码系列-003
tags:
  - sw
  - 软实现
cover: 'https://s2.loli.net/2022/11/12/SYeZ7wR48ajJUGh.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 49018
date: 2022-11-12 20:14:28
---

# `BCCSP`接口

`BCCSP`接口定义了以下方法，其实对密码学中的函数进行了一个功能分类：

- `KeyGen`：密钥生成，包含对称和非对称加密
- `KeyDeriv`：密钥派生
- `KeyImport`：密钥导入，从文件、内存、数字证书中导入
- `GetKey`：获取密钥
- `Hash`：计算摘要
- `GetHash`：获取摘要计算实例
- `Sign`：数字签名
- `Verify`：签名验证
- `Encrypt`：数据加密，包含对称和非对称加密
- `Decrypt`：数据解密，包含对称和非对称加密

# SW

## CSP

`fabric/bccsp/sw/impl.go`

`CSP`是一个`SW WRAPPER`，**调用`BCCSP`中定义的方法的总的入口就是`CSP`**, `CSP`利用反射调用真正的实现

```GO
type CSP struct {
	ks bccsp.KeyStore

	KeyGenerators map[reflect.Type]KeyGenerator
	KeyDerivers   map[reflect.Type]KeyDeriver
	KeyImporters  map[reflect.Type]KeyImporter
	Encryptors    map[reflect.Type]Encryptor
	Decryptors    map[reflect.Type]Decryptor
	Signers       map[reflect.Type]Signer
	Verifiers     map[reflect.Type]Verifier
	Hashers       map[reflect.Type]Hasher
}
```

**结构体可以使用反射调用`BCCSP`接口的具体实现，`KeyGenerators` 生成的如果`非临时KEY`，就会被保存在`KeyStore`中

`SW`中包含了`AES`、`ECDSA`、`HASH(SHA256)`、`RSA`, 对应于BCCSP接口中定义的方法，有以下实现：

- `KeyGen:` `fabric/bccsp/sw/keygen.go`

  包含了`ECDSA`、`AES`2种密钥的生成

  - `ecdsaKeyGenerator`
  - `aesKeyGenerator`
  
- `KeyDeriv:` `fabric/bccsp/sw/keyderiv.go`

  包含了`ECDSA`的公钥、私钥派生，`AES`的私钥派生 3种
  - `ecdsaPublicKeyKeyDeriver`
  - `ecdsaPrivateKeyKeyDeriver`
  - `aesPrivateKeyKeyDeriver`

- `KeyImport:` `fabric/bccsp/sw/keyimport.go`
  包含了:
  
  - `aes256ImportKeyOptsKeyImporter`
  - `hmacImportKeyOptsKeyImporter`
  - `ecdsaPKIXPublicKeyImportOptsKeyImporter`
  - `ecdsaPrivateKeyImportOptsKeyImporter`
  - `ecdsaGoPublicKeyImportOptsKeyImporter`
  - `x509PublicKeyImportOptsKeyImporter`

- `GetKey:` `KeyStore`的`GetKey`的调用，这里没有反射调用

- `Hash:` `fabric/bccsp/sw/hash.go`
  - `hasher `结构体实现了`Hash`方法
- `GetHash:` `fabric/bccsp/sw/hash.go`
  - `hasher `结构体实现了`GetHash`方法

- `Sign:` `fabric/bccsp/sw/ecdsa.go`
  - `ecdsaSigner`结构体实现了此方法，在`fabric 1.4`版中好像`rsaSigner`结构体也实现了此方法

- `Verify:` `fabric/bccsp/sw/ecdsa.go`
  - `ecdsaPrivateKeyVerifier` 结构体中实现了此方法
  - `ecdsaPublicKeyKeyVerifier`结构体中实现了此方法

- `Encrypt:` `fabric/bccsp/sw/aes.go`
  - `aescbcpkcs7Encryptor` 结构体
- `Decrypt:` `fabric/bccsp/sw/aes.go`
  - `aescbcpkcs7Encryptor` 结构体
