## 消息摘要算法

### 简介

消息摘要算法是一种通过对任意长度数据生成固定长度摘要来确保数据完整性的技术。常见的算法有MD5、SHA-1和SHA-256。

**应用场景**

消息摘要用于验证数据是否被篡改，例如在校验文件完整性、密码存储和数字签名中。

**核心原理**

消息摘要算法基于密码学散列函数，输入数据经过不可逆的变换生成摘要，即使输入数据微小变化，摘要也会大幅改变。

**特点与注意事项**

- 摘要长度固定，通常以十六进制表示。
- 同样的输入产生相同的摘要，但反之不成立。
- 由于不可逆性，不应将摘要用于恢复原始数据。
- 安全性取决于算法强度，更长的摘要通常更安全。

**选择摘要算法**

在实际应用中，应根据需要权衡速度和安全性，选择适合的摘要算法。较安全的算法如SHA-256通常优先于MD5和SHA-1。

### 核心类

通过MessageDigest和Mac类生成消息摘要和认证码，确保信息完整性。

**MessageDigest类**

- **角色：** 计算消息的摘要。
- **用法：** 选择摘要算法（例如MD5、SHA-256）并生成相应的摘要。

**Mac类**

- **角色：** 计算消息认证码（Message Authentication Code）。
- **用法：** 使用密钥和摘要算法共同生成消息认证码。

### MD5算法

**生成摘要**

```java
MessageDigest md5 = MessageDigest.getInstance("MD5");
byte[] message = "待生成摘要的文本".getBytes();
byte[] digest = md5.digest(message);
```

### SHA-1算法

**生成摘要**

```java
MessageDigest sha1 = MessageDigest.getInstance("SHA-1");
byte[] message = "待生成摘要的文本".getBytes();
byte[] digest = sha1.digest(message);
```

### SHA-256算法

**生成摘要**

```java
MessageDigest sha256 = MessageDigest.getInstance("SHA-256");
byte[] message = "待生成摘要的文本".getBytes();
byte[] digest = sha256.digest(message);
```

以上三种算法的核心步骤非常相似，它们主要的区别在于算法的内部结构和产生摘要的长度。在实际应用中，更安全的SHA-256通常优先于MD5和SHA-1，因为MD5和SHA-1已经存在已知的攻击和弱点。

### 消息认证码 (MAC)

使用Mac类和密钥共同生成消息认证码，可确保信息的完整性和认证。

**生成消息认证码**

```java
// 创建密钥
SecretKeySpec secretKey = new SecretKeySpec("密钥内容".getBytes(), "HmacSHA256");

// 获取Mac实例
Mac mac = Mac.getInstance("HmacSHA256");
mac.init(secretKey);

// 计算消息认证码
byte[] message = "待生成认证码的文本".getBytes();
byte[] authenticationCode = mac.doFinal(message);
```

**验证消息认证码**

```java
// 使用相同的密钥
SecretKeySpec secretKey = new SecretKeySpec("密钥内容".getBytes(), "HmacSHA256");

// 获取Mac实例
Mac mac = Mac.getInstance("HmacSHA256");
mac.init(secretKey);

// 计算原始消息的认证码
byte[] originalMessage = "待生成认证码的文本".getBytes();
byte[] originalAuthenticationCode = mac.doFinal(originalMessage);

// 假设收到的消息和认证码
byte[] receivedMessage = "待验证的文本".getBytes(); // 如果与原始消息相同，则验证通过
byte[] receivedAuthenticationCode;

// 使用相同的Mac实例重新计算收到的消息的认证码
receivedAuthenticationCode = mac.doFinal(receivedMessage);

// 比较两个认证码是否相同
if (Arrays.equals(originalAuthenticationCode, receivedAuthenticationCode)) {
    System.out.println("消息验证成功");
} else {
    System.out.println("消息验证失败");
}
```