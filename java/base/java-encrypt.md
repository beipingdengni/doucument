# 单向加密

## MD5、MD2、SHA-512、SHA-384、SHA-256、SHA-224、SHA-1

>MD5算法,返回的结果始终是32位
>
>SHA-512算法,返回的结果始终是128位
>
>SHA-384算法,返回的结果始终是96位
>
>SHA-256算法,返回的结果始终是64位
>
>SHA-224算法,返回的结果始终是56位
>
>SHA-1算法,返回的结果始终是40位

```
String data=""

MessageDigest md = MessageDigest.getInstance("SHA-512");
byte[] messageDigest = md.digest(data.getBytes()); // 加密
```



## HmacSHA256、HmacSHA512、HmacMD5

> HmacSHA1算法,返回的结果始终是40位
>
> HmacSHA256算法,返回的结果始终是64位
>
> HmacSHA512算法,返回的结果始终是128位
>
> HmacMD5算法,返回的结果始终是32位

```java

String key=""
String data="";
Mac mac = Mac.getInstance("HmacSHA256");
mac.init(new SecretKeySpec(key.getBytes("UTF-8"), "HmacSHA256"));
byte[] signData = mac.doFinal(data.getBytes("UTF-8"));
StringBuilder sb = new StringBuilder();
for (byte item : signData) {
  // 转为16进制
	sb.append(Integer.toHexString((item & 0xFF) | 0x100).substring(1, 3));
}
String result = sb.toString().toUpperCase();
```



# AES （对称加密）

## 随机密码

```
// AES支持三种长度的密钥：128位、192位、256位。
// 代码中这种就是128位的加密密钥，16字节 * 8位/字节 = 128位。
// 因为美国对软件出口的控制,默认只支持128位;要使用256则需另下bcprov-jdk的jar包替换jre\lib\security下的jar
String random = RandomStringUtils.random(16, "abcdefghijklmnopqrstuvwxyz1234567890");
```



### 增加参数，防止SecureRandom生成随机变化

> -Djava.security.egd=file:/dev/./urandom
>
> ES对称加解密， 相同key加密结果不一致，因为Linux的强随机数而导致，需要在 jvm 加如下启动参数

### 1、有随机数增加(类和方法)

```java
private static final String KEY_ALGORITHM = "AES";
private static final String CHARSET = "utf-8";

// 加密
public static String aesEncrypt(String content, String password) {
   String result = null;
   try {
     KeyGenerator kgen = KeyGenerator.getInstance(KEY_ALGORITHM);
     kgen.init(128, new SecureRandom(password.getBytes()));
     SecretKey secretKey = kgen.generateKey();
     byte[] enCodeFormat = secretKey.getEncoded();
     SecretKeySpec key = new SecretKeySpec(enCodeFormat, KEY_ALGORITHM);
     Cipher cipher = Cipher.getInstance(KEY_ALGORITHM); //
     byte[] byteContent = content.getBytes(CHARSET);
     cipher.init(Cipher.ENCRYPT_MODE, key); //
     byte[] bytes = cipher.doFinal(byteContent);
     result = byte2hex(bytes);
   } catch (Exception e) {
     e.printStackTrace();
   }
   return result;
 }

//  解密
public static String aesDecrypt(String content, String password) {
  String result = null;
  byte[] contentByte = hex2byte(content);
  try {
    KeyGenerator kgen = KeyGenerator.getInstance(KEY_ALGORITHM);
    kgen.init(128, new SecureRandom(password.getBytes()));
    SecretKey secretKey = kgen.generateKey();
    byte[] enCodeFormat = secretKey.getEncoded();
    SecretKeySpec key = new SecretKeySpec(enCodeFormat, KEY_ALGORITHM);
    Cipher cipher = Cipher.getInstance(KEY_ALGORITHM); //
    cipher.init(Cipher.DECRYPT_MODE, key); //
    byte[] bytes = cipher.doFinal(contentByte);
    result = new String(bytes); //
  } catch (Exception e) {
    e.printStackTrace();
  }
  return result;
}

// 字符转byte
private static byte[] hex2byte(String hexStr) {
  if (hexStr.length() < 1)
    return null;
  byte[] result = new byte[hexStr.length() / 2];
  for (int i = 0; i < hexStr.length() / 2; i++) {
    int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
    int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
    result[i] = (byte) (high * 16 + low);
  }
  return result;
}

// byte转16进制
private static String byte2hex(byte[] buf) {
  StringBuffer sb = new StringBuffer();
  for (int i = 0; i < buf.length; i++) {
    String hex = Integer.toHexString(buf[i] & 0xFF);
    if (hex.length() == 1) {
      hex = '0' + hex;
    }
    sb.append(hex.toUpperCase());
  }
  return sb.toString();
}

```

### 2、AESUtil.java 源码（ECB、CBC）

>  **加密：** AES加密 -> Base64加密 -> 密文
>
> **解密：** Base64解密 -> AES解密 -> 明文

```java
import org.apache.commons.lang3.RandomStringUtils;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.Base64Utils;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

/**
 * AES加密工具类
 *
 * @author ACGkaka
 * @since 2021-06-18 19:11:03
 */
public class AESUtil {
    // 日志相关
    private static final Logger LOGGER = LoggerFactory.getLogger(AESUtil.class);
    // 编码
    private static final String ENCODING = "UTF-8";
    // 算法定义
    private static final String AES_ALGORITHM = "AES";
    // 指定填充方式
    private static final String CIPHER_PADDING = "AES/ECB/PKCS5Padding";
    private static final String CIPHER_CBC_PADDING = "AES/CBC/PKCS5Padding";
    // 偏移量(CBC中使用，增强加密算法强度)
    private static final String IV_SEED = "1234567812345678";

    /**
     * AES加密
     * @param content 待加密内容
     * @param aesKey  密码
     * @return
     */
    public static String encrypt(String content, String aesKey){
        if(StringUtils.isBlank(content)){
            LOGGER.info("AES encrypt: the content is null!");
            return null;
        }
        //判断秘钥是否为16位
        if(StringUtils.isNotBlank(aesKey) && aesKey.length() == 16){
            try {
                //对密码进行编码
                byte[] bytes = aesKey.getBytes(ENCODING);
                //设置加密算法，生成秘钥
                SecretKeySpec skeySpec = new SecretKeySpec(bytes, AES_ALGORITHM);
                // "算法/模式/补码方式"
                Cipher cipher = Cipher.getInstance(CIPHER_PADDING);
                //选择加密
                cipher.init(Cipher.ENCRYPT_MODE, skeySpec);
                //根据待加密内容生成字节数组
                byte[] encrypted = cipher.doFinal(content.getBytes(ENCODING));
                //返回base64字符串
                return Base64Utils.encodeToString(encrypted);
            } catch (Exception e) {
                LOGGER.info("AES encrypt exception:" + e.getMessage());
                throw new RuntimeException(e);
            }

        }else {
            LOGGER.info("AES encrypt: the aesKey is null or error!");
            return null;
        }
    }

    /**
     * 解密
     * 
     * @param content 待解密内容
     * @param aesKey  密码
     * @return
     */
    public static String decrypt(String content, String aesKey){
        if(StringUtils.isBlank(content)){
            LOGGER.info("AES decrypt: the content is null!");
            return null;
        }
        //判断秘钥是否为16位
        if(StringUtils.isNotBlank(aesKey) && aesKey.length() == 16){
            try {
                //对密码进行编码
                byte[] bytes = aesKey.getBytes(ENCODING);
                //设置解密算法，生成秘钥
                SecretKeySpec skeySpec = new SecretKeySpec(bytes, AES_ALGORITHM);
                // "算法/模式/补码方式"
                Cipher cipher = Cipher.getInstance(CIPHER_PADDING);
                //选择解密
                cipher.init(Cipher.DECRYPT_MODE, skeySpec);

                //先进行Base64解码
                byte[] decodeBase64 = Base64Utils.decodeFromString(content);

                //根据待解密内容进行解密
                byte[] decrypted = cipher.doFinal(decodeBase64);
                //将字节数组转成字符串
                return new String(decrypted, ENCODING);
            } catch (Exception e) {
                LOGGER.info("AES decrypt exception:" + e.getMessage());
                throw new RuntimeException(e);
            }

        }else {
            LOGGER.info("AES decrypt: the aesKey is null or error!");
            return null;
        }
    }

    /**
     * AES_CBC加密
     * 
     * @param content 待加密内容
     * @param aesKey  密码
     * @return
     */
    public static String encryptCBC(String content, String aesKey){
        if(StringUtils.isBlank(content)){
            LOGGER.info("AES_CBC encrypt: the content is null!");
            return null;
        }
        //判断秘钥是否为16位
        if(StringUtils.isNotBlank(aesKey) && aesKey.length() == 16){
            try {
                //对密码进行编码
                byte[] bytes = aesKey.getBytes(ENCODING);
                //设置加密算法，生成秘钥
                SecretKeySpec skeySpec = new SecretKeySpec(bytes, AES_ALGORITHM);
                // "算法/模式/补码方式"
                Cipher cipher = Cipher.getInstance(CIPHER_CBC_PADDING);
                //偏移
                IvParameterSpec iv = new IvParameterSpec(IV_SEED.getBytes(ENCODING));
                //选择加密
                cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv);
                //根据待加密内容生成字节数组
                byte[] encrypted = cipher.doFinal(content.getBytes(ENCODING));
                //返回base64字符串
                return Base64Utils.encodeToString(encrypted);
            } catch (Exception e) {
                LOGGER.info("AES_CBC encrypt exception:" + e.getMessage());
                throw new RuntimeException(e);
            }

        }else {
            LOGGER.info("AES_CBC encrypt: the aesKey is null or error!");
            return null;
        }
    }

    /**
     * AES_CBC解密
     * 
     * @param content 待解密内容
     * @param aesKey  密码
     * @return
     */
    public static String decryptCBC(String content, String aesKey){
        if(StringUtils.isBlank(content)){
            LOGGER.info("AES_CBC decrypt: the content is null!");
            return null;
        }
        //判断秘钥是否为16位
        if(StringUtils.isNotBlank(aesKey) && aesKey.length() == 16){
            try {
                //对密码进行编码
                byte[] bytes = aesKey.getBytes(ENCODING);
                //设置解密算法，生成秘钥
                SecretKeySpec skeySpec = new SecretKeySpec(bytes, AES_ALGORITHM);
                //偏移
                IvParameterSpec iv = new IvParameterSpec(IV_SEED.getBytes(ENCODING));
                // "算法/模式/补码方式"
                Cipher cipher = Cipher.getInstance(CIPHER_CBC_PADDING);
                //选择解密
                cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);

                //先进行Base64解码
                byte[] decodeBase64 = Base64Utils.decodeFromString(content);

                //根据待解密内容进行解密
                byte[] decrypted = cipher.doFinal(decodeBase64);
                //将字节数组转成字符串
                return new String(decrypted, ENCODING);
            } catch (Exception e) {
                LOGGER.info("AES_CBC decrypt exception:" + e.getMessage());
                throw new RuntimeException(e);
            }

        }else {
            LOGGER.info("AES_CBC decrypt: the aesKey is null or error!");
            return null;
        }
    }
  	
  	// 数字和26个字母组成
		private static final String SYMBOLS = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"; 
		private static final Random RANDOM = new SecureRandom();

		// 获取长度为 6 的随机字母+数字
    public static String getRandomNumber() {
        char[] nonceChars = new char[16];  //指定长度为6位/自己可以要求设置

        for (int index = 0; index < nonceChars.length; ++index) {
            nonceChars[index] = SYMBOLS.charAt(RANDOM.nextInt(SYMBOLS.length()));
        }
        return new String(nonceChars);
    }

    public static void main(String[] args) {
        // AES支持三种长度的密钥：128位、192位、256位。
        // 代码中这种就是128位的加密密钥，16字节 * 8位/字节 = 128位。
        String random = RandomStringUtils.random(16, "abcdefghijklmnopqrstuvwxyz1234567890");
        System.out.println("随机key:" + random);
        System.out.println();

        System.out.println("---------加密---------");
        String aesResult = encrypt("测试AES加密12", random);
        System.out.println("aes加密结果:" + aesResult);
        System.out.println();

        System.out.println("---------解密---------");
        String decrypt = decrypt(aesResult, random);
        System.out.println("aes解密结果:" + decrypt);
        System.out.println();


        System.out.println("--------AES_CBC加密解密---------");
        String cbcResult = encryptCBC("测试AES加密12456", random);
        System.out.println("aes_cbc加密结果:" + cbcResult);
        System.out.println();

        System.out.println("---------解密CBC---------");
        String cbcDecrypt = decryptCBC(cbcResult, random);
        System.out.println("aes解密结果:" + cbcDecrypt);
        System.out.println();
    }
}

```



# DES

