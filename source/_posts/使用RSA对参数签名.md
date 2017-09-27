---
title: 使用RSA对参数签名
date: 2017-09-26
tags: [RSA]
categories: [加密]
---

## 导论
记录最近使用最多的RSA签名加密。由于所在行业安全要求比较高，请求的参数都是加密过后再使用的，使用的是RSA加密算法。需要请求方讲参数使用私钥加密，处理方获取到参数后需要使用公钥验签，通过再处理业务逻辑，这是最简单的加密和解密。

笔者使用的是OpenSSL工具生成的公钥和私钥暂存文件，签名和验签使用的是jdk自行支持的签名算法。

## 正文

### 使用OpenSSL先生成公钥和私钥的暂存文件
首先下载OpenSSL的工具类，搞完之后直接执行指令之后即可。[参考地址](https://docs.open.alipay.com/291/106130)

执行指令如下图:

![image](http://otqvaruzt.bkt.clouddn.com/0926_01.png)

执行之后会生成三个pem文件：

rsa_public_key.pem：公钥文件
rsa_private_key.pem：私钥文件
rsa_private_key_pkcs8.pem：pkc格式的私钥文件，java专用。

所有的准备工作都ok了，接下来来测试

### 测试
测试代码

```
package com.mxl.rsa.demo;

import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SignatureException;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

import org.junit.Test;

/**
 * 
 * @author MXL
 *
 */
public class TestSign {
	
	private static final String PRIVATE_PATH = "rsa_private_key_pkcs8.pem";
	
	private static final String PUBLIC_PATH = "rsa_public_key.pem";
	
	@Test
	public void testSign() throws InvalidKeyException, NoSuchAlgorithmException, SignatureException{
		//获取私钥路径
		String privateKeyFile = TestSign.class.getClassLoader().getResource("").getPath() + PRIVATE_PATH;
		//获取公钥路径
		String publicKeyFile = TestSign.class.getClassLoader().getResource("").getPath() + PUBLIC_PATH;
		
		privateKeyFile = privateKeyFile.replaceAll("%20", " ");
		publicKeyFile = publicKeyFile.replaceAll("%20", " ");
		
		//使用JDK自带的方式获取私钥
		RSAPrivateKey rsaPrivateKey = RSAUtils.initPemPrivateKey(privateKeyFile);
		//公钥获取
		RSAPublicKey rsaPublicKey = RSAUtils.initPemPublicKey(publicKeyFile);
		//构造签名内容
		String signData = "Hello RSA";
		//使用私钥签名
		byte[] signature = RSAUtils.sign(rsaPrivateKey, signData.getBytes());
		
		if(RSAUtils.verifySignature(rsaPublicKey, signData.getBytes(), signature)){
			System.out.println("验签成功，是自己人");
		}else {
			System.out.println("验签失败");
		}
	}
	
}

outPut：

验签成功，是自己人
```
代码每一行都加了注释，很好理解，下面来看看RSAUtils中的内容。

获取私钥
```
public static RSAPrivateKey initPemPrivateKey(String privateKeyFile) {  
    BufferedReader br = null;
    try {  
        br = new BufferedReader(new FileReader(privateKeyFile));  
        String s = br.readLine();  
        StringBuffer privatekey = new StringBuffer();  
        s = br.readLine();  
        while (s.charAt(0) != '-') {  
            privatekey.append(s + "\r");  
            s = br.readLine();  
        }  
              
        byte[] keybyte = new BASE64Decoder().decodeBuffer(privatekey.toString());  
              
        KeyFactory kf = KeyFactory.getInstance("RSA");  
              
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keybyte);  
        return (RSAPrivateKey)kf.generatePrivate(keySpec);  
    } catch (Exception e) {  
        e.printStackTrace();
    } finally{
		try {
			if(br!=null)
				br.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
    }
    return null;
} 
```
代码好理解，这里不废话了，获取公钥也是类似

签名
```
public static byte[] sign(RSAPrivateKey privateKey, byte[] b) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
	Signature sig = Signature.getInstance("SHA1withRSA"); 
	sig.initSign(privateKey);
	sig.update(b);
	byte[] signature = sig.sign();
	return signature;
}
```

验签
```
public static boolean verifySignature(RSAPublicKey publicKey, byte[] data, byte[] signature)
	throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
	Signature sig2 = Signature.getInstance("SHA1withRSA");
	sig2.initVerify(publicKey);
	sig2.update(data);
	return sig2.verify(signature);
}
```

## 总结
参数加密在支付和javaCard中使用的是很频繁的，记录归档，方便复习。另外在网上看到一句不错的总结：私钥加密，公钥解密；私钥签名，公钥验签。
