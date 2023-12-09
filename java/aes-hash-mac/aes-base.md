



说明：AES、DES   https://blog.csdn.net/apr15/article/details/126680015

密码学在Java中的实现：使用crypto包进行安全编程    https://zhuanlan.zhihu.com/p/649677500



## AES

### CBC

```java
String tokenStr="16字母字符";    //defaultToken = "T$2#*!Sd*134_)8i";
String ivParam="iv向量16字符"

  // 1 加密、2 解密
int mode=Cipher.ENCRYPT_MODE;  //  加密
int mode=Cipher.DECRYPT_MODE;   // 解密

byte[] token=tokenStr.getBytes(StandardCharsets.UTF_8)
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5PADDING");
Key key = new SecretKeySpec(token, "AES");
AlgorithmParameters params = AlgorithmParameters.getInstance("AES");
params.init(new IvParameterSpec(ivParam.getBytes(StandardCharsets.UTF_8)));
cipher.init(mode, key, params);
// 解密
byte[] content=Base64.decodeBase64("内容")
byte[]  result = cipher.doFinal(content);
new String(result, StandardCharsets.UTF_8);
// 加密  
byte[] content.getBytes(StandardCharsets.UTF_8)
byte[]  result = cipher.doFinal(content);
Base64.encodeBase64URLSafeString(result)
// Base64  来自 commons-codec:commons-codec:1.13

```

### 在线使用

在线使用：https://the-x.cn/cryptography/Aes.aspx     aes cbc模式 pkcs7  key及iv是 = "16位字符位密码和iv"

# SecureRandom

> securerandom.source=file:/dev/random   滚定生成的不会变化

```java
import org.apache.commons.codec.binary.Hex;
import org.apache.commons.lang3.StringUtils;
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.Key;
import java.security.SecureRandom;
 
public class AESUtil {
    private static final String KEY_ALGORITHM = "AES";
    private static final String DEFAULT_CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";
 
    /**
     * 指定随机字符串（密码）生成密钥
     *
     * @param randomKey 加解密的密码
     * @throws Exception
     */
    public static byte[] getSecretKey(String randomKey) throws Exception {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_ALGORITHM); //秘钥生成器，指定秘钥算法
 
        //初始化此密钥生成器，指定AES的秘钥长度为128
        if (StringUtils.isBlank(randomKey)) {   //不指定密码
            keyGenerator.init(128);
        } else {    //指定密码
            SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
            random.setSeed(randomKey.getBytes());
            keyGenerator.init(128, random);
        }
 
        SecretKey secretKey = keyGenerator.generateKey();   //生成密钥
        return secretKey.getEncoded();
    }
 
    /**
     * 加密
     *
     * @param data 待加密数据
     * @param key  密钥
     * @return byte[]   加密数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, Key key) throws Exception {
        return encrypt(data, key, DEFAULT_CIPHER_ALGORITHM);
    }
 
    /**
     * 加密
     *
     * @param data 待加密数据
     * @param key  二进制密钥
     * @return byte[]   加密数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, byte[] key) throws Exception {
        return encrypt(data, key, DEFAULT_CIPHER_ALGORITHM);
    }
 
    /**
     * 加密
     *
     * @param data            待加密数据
     * @param key             二进制密钥
     * @param cipherAlgorithm 加密算法/工作模式/填充方式
     * @return byte[]   加密数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, byte[] key, String cipherAlgorithm) throws Exception {
        Key k = toKey(key);
        return encrypt(data, k, cipherAlgorithm);
    }
 
    /**
     * 加密
     *
     * @param data            待加密数据
     * @param key             密钥
     * @param cipherAlgorithm 加密算法/工作模式/填充方式
     * @return byte[]   加密数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, Key key, String cipherAlgorithm) throws Exception {
        Cipher cipher = Cipher.getInstance(cipherAlgorithm);    //获取算法
        cipher.init(Cipher.ENCRYPT_MODE, key);                  //设置加密模式，并指定秘钥
        return cipher.doFinal(data);                            //加密数据
    }
 
    /**
     * 解密
     *
     * @param data 待解密数据
     * @param key  二进制密钥
     * @return byte[]   解密数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, byte[] key) throws Exception {
        return decrypt(data, key, DEFAULT_CIPHER_ALGORITHM);
    }
 
    /**
     * 解密
     *
     * @param data 待解密数据
     * @param key  密钥
     * @return byte[]   解密数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, Key key) throws Exception {
        return decrypt(data, key, DEFAULT_CIPHER_ALGORITHM);
    }
 
    /**
     * 解密
     *
     * @param data            待解密数据
     * @param key             二进制密钥
     * @param cipherAlgorithm 加密算法/工作模式/填充方式
     * @return byte[]   解密数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, byte[] key, String cipherAlgorithm) throws Exception {
        Key k = toKey(key);
        return decrypt(data, k, cipherAlgorithm);
    }
 
    /**
     * 解密
     *
     * @param data            待解密数据
     * @param key             密钥
     * @param cipherAlgorithm 加密算法/工作模式/填充方式
     * @return byte[]   解密数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, Key key, String cipherAlgorithm) throws Exception {
        Cipher cipher = Cipher.getInstance(cipherAlgorithm);     //获取算法
        cipher.init(Cipher.DECRYPT_MODE, key);                   //设置解密模式，并指定秘钥
        return cipher.doFinal(data);                             //解密数据
    }
 
    /**
     * 转换密钥
     *
     * @param secretKey 二进制密钥
     * @return 密钥
     */
    public static Key toKey(byte[] secretKey) {
        return new SecretKeySpec(secretKey, KEY_ALGORITHM);   //生成密钥
    }
 
 
    public static void main(String[] args) throws Exception {
        String password = "123456";     //加解密的密码
        byte[] secretKey = getSecretKey(password);
        Key key = toKey(secretKey);
 
        String data = "AES 对称加密算法";
        System.out.println("明文 ：" + data);
 
        byte[] encryptData = encrypt(data.getBytes(), key);
        String encryptDataHex = Hex.encodeHexString(encryptData);   //把密文转为16进制
        System.out.println("加密 : " + encryptDataHex);
 
        byte[] decryptData = decrypt(Hex.decodeHex(encryptDataHex.toCharArray()), key);
        System.out.println("解密 : " + new String(decryptData));
    }
}
```

