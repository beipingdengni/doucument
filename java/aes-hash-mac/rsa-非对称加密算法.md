## 非对称加密算法

### 简介

非对称加密算法使用一对密钥进行加密和解密，分别是公钥和私钥，常见的算法有RSA、DSA和ECC。

**应用场景**

非对称加密在身份验证、数字签名和密钥交换中扮演重要角色，实现了更高级别的数据安全性。

**加密与解密**

使用公钥进行加密，只有持有对应私钥的一方能解密。私钥用于签名和解密。

**密钥交换**

非对称加密算法也用于密钥交换协议，例如Diffie-Hellman密钥交换，安全地协商对称加密的密钥。

**数字签名**

数字签名通过私钥对数据进行签名，公钥用于验证签名的真实性，确保数据完整性和认证。

**强度与计算**

与对称加密相比，非对称加密强度更高，但也更消耗计算资源，通常用于较小数据量和关键通信。

### 核心类

**KeyPair类**

- **角色：** 非对称密钥对。
- **用法：** 用于加密和解密。

**KeyPairGenerator类**

- **角色：** 生成非对称密钥对。
- **用法：** 选择非对称算法（例如RSA）并生成公钥和私钥。

**Cipher类**

- **角色：** 执行非对称加密和解密。
- **用法：** 使用公钥进行加密，私钥进行解密。

### RSA加密

**创建密钥对**

```java
KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
keyPairGen.initialize(2048);
KeyPair keyPair = keyPairGen.generateKeyPair();
```

**初始化Cipher对象**

```java
Cipher cipher = Cipher.getInstance("RSA");
cipher.init(Cipher.ENCRYPT_MODE, keyPair.getPublic());
```

**执行加密**

```java
byte[] cipherText = cipher.doFinal(plainText);
```

**执行解密**

```java
cipher.init(Cipher.DECRYPT_MODE, keyPair.getPrivate());
byte[] decryptedText = cipher.doFinal(cipherText);
```