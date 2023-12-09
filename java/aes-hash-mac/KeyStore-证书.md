## KeyStore

### 简介

KeyStore是Java安全框架中的一部分，用于管理密钥和证书。

**KeyStore类型（如"JKS"）**

KeyStore类型定义了密钥库的存储格式。"JKS"是Java KeyStore的缩写，是Java的专有类型。Java 9之后推荐使用"PKCS12"，它是一个广泛接受的标准格式。

**密钥库文件**

密钥库文件是一个包含密钥和证书的物理文件。你可以将其视为一个安全的容器，用于存储私钥、公钥、证书等。

**密钥库密码**

为了保护密钥库文件的完整性，你需要为其设置一个密码。这个密码用于确保只有知道密码的人才能访问密钥库。

**密钥别名**

在密钥库中，每个密钥都有一个唯一的别名，可以用这个别名来引用密钥。你可以将其视为密钥的"用户名"，用于在密钥库中找到特定的密钥。

**密钥密码**

除了密钥库密码，每个私钥还可以有自己的密码。这提供了另一层保护，确保只有知道此特定密码的人才能访问该私钥。

**证书别名**

与密钥别名类似，证书别名用于在密钥库中引用特定的证书。

**证书**

证书是一个数字文档，用于证明公钥的所有权。证书由受信任的证书颁发机构（CA）签名，并可以用于验证公钥的真实性。

**实际使用**

在实际使用中，开发者可以使用KeyStore来安全地存储和管理密钥和证书。例如，可以在服务器上存储私钥，并使用它来进行TLS握手。或者，可以在客户端上存储公钥证书，并使用它来验证服务器的身份。

### 核心类

**KeyStore类**

- **角色：** 管理密钥和证书。
- **用法：** 从文件中加载密钥和证书，或将密钥和证书保存到文件中。

### 保存密钥和证书

```java
// 生成固定种子的随机数源，用于确保密钥对生成的可重复性
SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
secureRandom.setSeed(12345L);

// 使用RSA算法和固定种子初始化密钥对生成器
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
keyPairGenerator.initialize(2048, secureRandom);
KeyPair keyPair = keyPairGenerator.generateKeyPair(); // 生成密钥对

// 创建证书信息对象，并设置有效期、序列号、拥有者等基本信息
X509CertInfo info = new X509CertInfo();
Date from = new Date();
Date to = new Date(from.getTime() + 365 * 86400000L);
CertificateValidity interval = new CertificateValidity(from, to);
X500Name owner = new X500Name("CN=localhost");
info.set(X509CertInfo.VALIDITY, interval);
info.set(X509CertInfo.SERIAL_NUMBER, new CertificateSerialNumber((int) (from.getTime() / 1000)));
info.set(X509CertInfo.SUBJECT, owner);
info.set(X509CertInfo.ISSUER, owner);
info.set(X509CertInfo.KEY, new CertificateX509Key(keyPair.getPublic()));
info.set(X509CertInfo.VERSION, new CertificateVersion(CertificateVersion.V3));
AlgorithmId algo = new AlgorithmId(AlgorithmId.md5WithRSAEncryption_oid);
info.set(X509CertInfo.ALGORITHM_ID, new CertificateAlgorithmId(algo));

// 创建并签名X509证书
X509CertImpl certImpl = new X509CertImpl(info);
certImpl.sign(keyPair.getPrivate(), "MD5withRSA");

// 创建密钥库并保存密钥和证书
KeyStore keyStore = KeyStore.getInstance("JKS");
keyStore.load(null, null);
keyStore.setKeyEntry("my_key", keyPair.getPrivate(), "key_password".toCharArray(), new java.security.cert.Certificate[]{certImpl});
keyStore.setCertificateEntry("my_certificate", certImpl);
keyStore.store(new FileOutputStream("keystore.jks"), "keystore_password".toCharArray()); // 保存到文件
```

### 加载密钥和证书

```java
// 从文件加载JKS密钥库
KeyStore keyStore = KeyStore.getInstance("JKS");
keyStore.load(new FileInputStream("keystore.jks"), "keystore_password".toCharArray());

// 从密钥库加载私钥和证书
Key key = keyStore.getKey("my_key", "key_password".toCharArray());
java.security.cert.Certificate certificate = keyStore.getCertificate("my_certificate");

// 断言私钥和证书的类型、算法和值
assertThat(key).isInstanceOf(PrivateKey.class);
assertThat(key.getAlgorithm()).isEqualTo("RSA");
assertThat(Base64.getEncoder().encodeToString(key.getEncoded())).isEqualTo(KEYSTORE_PRIVATE_KEY);
assertThat(certificate.getType()).isEqualTo("X.509");

// 从证书中获取公钥并断言
PublicKey publicKey = certificate.getPublicKey();
assertThat(publicKey.getAlgorithm()).isEqualTo("RSA");
assertThat(Base64.getEncoder().encodeToString(publicKey.getEncoded())).isEqualTo(KEYSTORE_PUBLIC_KEY);

// 签名和验证，确保公钥和私钥匹配
Signature signature = Signature.getInstance("MD5withRSA");
byte[] data = "test data".getBytes(); // 待签名的数据
signature.initSign((PrivateKey) key); // 使用私钥签名
signature.update(data);
byte[] digitalSignature = signature.sign(); // 计算签名
signature.initVerify(publicKey); // 使用公钥验证
signature.update(data);
assertThat(signature.verify(digitalSignature)).isTrue(); // 验证签名
```

## 证书

### 简介

证书是一个由可信任的实体颁发的电子文件，用于证明证书持有者的身份以及其公钥的真实性。证书可以有多种类型，其中X.509证书是最常见的一种。

**为什么需要证书？**

1. **身份验证**：证书通过证书颁发机构的签名确保了持有者身份的真实性和可信度。
2. **保障通信安全**：证书中的公钥用于加密和数字签名，确保通信的保密性和完整性。

**证书的组成部分**

1. **主体信息**：包括持有者的名称、组织等信息。
2. **公钥信息**：持有者的公钥。
3. **签名算法**：用于证书签名的算法。
4. **颁发机构信息**：证书颁发机构（CA）的信息。
5. **有效期**：证书的有效时间范围。
6. **数字签名**：由证书颁发机构使用其私钥创建的签名。

**证书颁发机构（CA）**

证书颁发机构是可信任的组织，负责验证申请者的身份并签署其证书。主流的浏览器和操作系统通常会预安装一些知名的CA根证书。

**常用的证书类型**

1. **X.509证书**：最常见的证书类型，广泛用于SSL/TLS等。
2. **PGP证书**：用于电子邮件的加密和身份验证。

**证书在哪里使用？**

1. **HTTPS网站**：使用SSL/TLS证书保护用户与服务器之间的通信。
2. **电子签名**：用于验证电子文档的完整性和签名者的身份。

**如何查看网站的证书？**

在大多数现代浏览器中，你可以点击地址栏的锁符号，然后选择“证书”来查看访问网站的证书信息。

### 证书和数字证书的区别

数字证书是一种特定类型的证书，常指遵循X.509国际标准的证书。

**数字证书（X509Certificate）**

- **定义：** 遵循X.509国际标准的证书。
- **内容：** 包括公钥、证书持有者信息、有效期、签名算法、证书颁发机构（CA）的信息和数字签名等。
- **格式：** 严格的X.509格式规定。
- **用途：** 用于身份验证、数据加密、数字签名等。常用于HTTPS、电子签名和加密邮件等场景。

**总结**

- **证书**是一个更通用的概念，可以包括各种格式和类型。
- **数字证书**则是一种特定的证书，遵循X.509标准，并用于特定的网络安全场景。

### 核心类

**Certificate类**

- **角色：** 证书。
- **用法：** 证书包含公钥和证书持有者的信息，可以用于验证签名。

**X509Certificate类**

- **角色：** 数字证书。
- **用法：** 数字证书包含公钥和证书持有者的信息，可以用于验证签名。

### 保存证书

```java
// 生成固定种子的随机数源，用于确保密钥对生成的可重复性
SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
secureRandom.setSeed(12345L);

// 使用RSA算法和固定种子初始化密钥对生成器
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
keyPairGenerator.initialize(2048, secureRandom);
KeyPair keyPair = keyPairGenerator.generateKeyPair(); // 生成密钥对

// 创建证书信息对象，并设置有效期、序列号、拥有者等基本信息
X509CertInfo info = new X509CertInfo();
Date from = new Date();
Date to = new Date(from.getTime() + 365 * 86400000L);
CertificateValidity interval = new CertificateValidity(from, to);
X500Name owner = new X500Name("CN=localhost");
info.set(X509CertInfo.VALIDITY, interval);
info.set(X509CertInfo.SERIAL_NUMBER, new CertificateSerialNumber((int) (from.getTime() / 1000)));
info.set(X509CertInfo.SUBJECT, owner);
info.set(X509CertInfo.ISSUER, owner);
info.set(X509CertInfo.KEY, new CertificateX509Key(keyPair.getPublic()));
info.set(X509CertInfo.VERSION, new CertificateVersion(CertificateVersion.V3));
AlgorithmId algo = new AlgorithmId(AlgorithmId.md5WithRSAEncryption_oid);
info.set(X509CertInfo.ALGORITHM_ID, new CertificateAlgorithmId(algo));

// 创建并签名X509证书
X509CertImpl certImpl = new X509CertImpl(info);
certImpl.sign(keyPair.getPrivate(), "MD5withRSA");

// 将证书保存到文件
try (FileOutputStream fos = new FileOutputStream("certificate.cer")) {
    fos.write(certImpl.getEncoded());
}
```

### 加载证书

```java
// 创建证书工厂
CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");

// 从文件加载证书
try (FileInputStream fis = new FileInputStream("certificate.cer")) {
    java.security.cert.Certificate certificate = certificateFactory.generateCertificate(fis);

    // 断言证书类型
    assertThat(certificate.getType()).isEqualTo("X.509");

    // 从证书中获取公钥
    PublicKey publicKey = certificate.getPublicKey();
    // 断言公钥算法
    assertThat(publicKey.getAlgorithm()).isEqualTo("RSA");
    // 断言公钥值
    assertThat(Base64.getEncoder().encodeToString(publicKey.getEncoded())).isEqualTo(KEYSTORE_PUBLIC_KEY);
}
```

### 保存数字证书

```java
// 创建密钥对
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
keyPairGenerator.initialize(2048);
KeyPair keyPair = keyPairGenerator.generateKeyPair();

// 创建证书主体
X500Name dnName = new X500Name("CN=localhost", "OU=OrgUnit", "O=Org", "L=City", "ST=State", "C=Country");
BigInteger certSerialNumber = new BigInteger(Long.toString(System.currentTimeMillis()));

// 证书有效期
long validity = 365 * 24 * 60 * 60 * 1000; // 1 year
Date startDate = new Date();
Date endDate = new Date(startDate.getTime() + validity);

// 创建证书信息
X509CertInfo certInfo = new X509CertInfo();
certInfo.set(X509CertInfo.VALIDITY, new CertificateValidity(startDate, endDate));
certInfo.set(X509CertInfo.SERIAL_NUMBER, new CertificateSerialNumber(certSerialNumber));
certInfo.set(X509CertInfo.SUBJECT, dnName); // 使用 X500Name
certInfo.set(X509CertInfo.ISSUER, dnName);  // 使用 X500Name
certInfo.set(X509CertInfo.KEY, new CertificateX509Key(keyPair.getPublic()));
certInfo.set(X509CertInfo.VERSION, new CertificateVersion(CertificateVersion.V3));
AlgorithmId algoId = new AlgorithmId(AlgorithmId.sha256WithRSAEncryption_oid);
certInfo.set(X509CertInfo.ALGORITHM_ID, new CertificateAlgorithmId(algoId));

// 创建并签名证书
X509CertImpl certImpl = new X509CertImpl(certInfo);
certImpl.sign(keyPair.getPrivate(), "SHA256withRSA");

// 将证书保存到文件
try (FileOutputStream fos = new FileOutputStream("x509_certificate.cer")) {
    fos.write(certImpl.getEncoded());
}
```

### 加载数字证书

```java
// 创建证书工厂
CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");

// 从文件加载证书
try (FileInputStream fis = new FileInputStream("x509_certificate.cer")) {
    X509Certificate certificate = (X509Certificate) certificateFactory.generateCertificate(fis);

    // 断言证书类型
    assertThat(certificate.getType()).isEqualTo("X.509");

    // 从证书中获取公钥
    PublicKey publicKey = certificate.getPublicKey();
    // 断言公钥算法
    assertThat(publicKey.getAlgorithm()).isEqualTo("RSA");
    // 断言公钥值
    assertThat(Base64.getEncoder().encodeToString(publicKey.getEncoded())).isEqualTo(X509_PUBLIC_KEY);
}
```