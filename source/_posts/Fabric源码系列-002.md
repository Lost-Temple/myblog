---
title: Fabric源码系列-002
tags:
  - BCCSP
  - SW
  - PKCS11
  - 依赖倒置原则
  - 密码学插件
  - Plugin
cover: 'https://s2.loli.net/2022/11/12/SYeZ7wR48ajJUGh.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 32699
date: 2022-11-12 09:42:56
---

# BCCSP 接口

**以下内容从[太彬年久失修的博客](https://lessisbetter.site/)盗版而来**

`BCCSP`是`Block Chain Crypto Service Provider`的缩写。在Fabric中，BCCSP被抽象成为一个接口。

`bccsp`模块提供密码学服务，它包含的具体功能有：对称加密和非对称加密的密钥生成、导入、导出，数字签名和验证，对称加密和解密、摘要计算。

`bccsp`使用了依赖倒置原则（Dependence Inversion Principle）：依赖抽象而不依赖具体实现，这样的好处就是可以把具体的密码学实现（实现BCCSP接口）当成插件进行使用。

bccsp模块中当前有2种密码实现，它们都是bccsp中的密码学插件：`SW`和`PKCS11`，`SW`代表的是国际标准加密的软实现，`SW`是`software`的缩写，`PKCS11`代指硬实现。

![BCCSP.png](https://s2.loli.net/2022/11/12/SYeZ7wR48ajJUGh.png)

> 扩展阅读：
>
> PKCS11是PKCS系列标准中的第11个，它定义了应用层和底层加密设备的交互标准，比如过去在电脑上，
> 插入USBKey用网银转账时，就需要走USBKey中的硬件进行数字签名，这个过程就需要使用PCKS11。

密码学通常有软实现和硬实现，软实现就是常用的各种加密库，比如Go中`crypto`包，硬实现是使用加密机提供的一套加密服务。软实现和硬实现的重要区别是，密码算法的安全性强依赖随机数，软实现利用的是OS的伪随机数，而硬实现利用的是加密机生成的随机数，所以硬实现的安全强度要高于软实现。

## SW介绍

SW是国际标准加密的软实现插件，它包含了ECDSA算法、RSA算法、AES算法，以及SHA系列的摘要算法。

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

```go
// BCCSP is the blockchain cryptographic service provider that offers
// the implementation of cryptographic standards and algorithms.
type BCCSP interface {

	// KeyGen generates a key using opts.
	KeyGen(opts KeyGenOpts) (k Key, err error)

	// KeyDeriv derives a key from k using opts.
	// The opts argument should be appropriate for the primitive used.
	KeyDeriv(k Key, opts KeyDerivOpts) (dk Key, err error)

	// KeyImport imports a key from its raw representation using opts.
	// The opts argument should be appropriate for the primitive used.
	KeyImport(raw interface{}, opts KeyImportOpts) (k Key, err error)

	// GetKey returns the key this CSP associates to
	// the Subject Key Identifier ski.
	GetKey(ski []byte) (k Key, err error)

	// Hash hashes messages msg using options opts.
	// If opts is nil, the default hash function will be used.
	Hash(msg []byte, opts HashOpts) (hash []byte, err error)

	// GetHash returns and instance of hash.Hash using options opts.
	// If opts is nil, the default hash function will be returned.
	GetHash(opts HashOpts) (h hash.Hash, err error)

	// Sign signs digest using key k.
	// The opts argument should be appropriate for the algorithm used.
	//
	// Note that when a signature of a hash of a larger message is needed,
	// the caller is responsible for hashing the larger message and passing
	// the hash (as digest).
	Sign(k Key, digest []byte, opts SignerOpts) (signature []byte, err error)

	// Verify verifies signature against key k and digest
	// The opts argument should be appropriate for the algorithm used.
	Verify(k Key, signature, digest []byte, opts SignerOpts) (valid bool, err error)

	// Encrypt encrypts plaintext using key k.
	// The opts argument should be appropriate for the algorithm used.
	Encrypt(k Key, plaintext []byte, opts EncrypterOpts) (ciphertext []byte, err error)

	// Decrypt decrypts ciphertext using key k.
	// The opts argument should be appropriate for the algorithm used.
	Decrypt(k Key, ciphertext []byte, opts DecrypterOpts) (plaintext []byte, err error)
}
```



## 可插拔国密

Fabric支持国密并非仅仅在bccsp中增加1个国密实现这么简单，还需要让数字证书支持国密，让数字证书的操作符合X.509。各语言的标准库`x509`都是适配标准加密的，并不能直接用来操作国密证书。

在数字证书支持国密后，还可能需要进一步考虑，是否需要TLS证书使用国密数字证书，让通信过程使用国密算法。

另外，国密的实现有很多版本，如果需要适配不同的国密实现，就需要保证国密的可插拔和可扩展。
