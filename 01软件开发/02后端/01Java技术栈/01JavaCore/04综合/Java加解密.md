static {
        Security.addProvider(new BouncyCastleProvider());
    }代码加这个试试

http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html


[使 JDK 支持 TLS_RSA_WITH_AES_256_CBC_SHA256 加密套件](http://blog.csdn.net/troylemon/article/details/51025929)

[](http://www.jianshu.com/p/733c6da85422)



密钥管理系统SOA2.0的接口使用的https，各BU接入前先确认一下生产服务器是否已装有CTRIP的根证书，如果没有安装证书，调用时会出现如下异常：
System.Net.WebException--The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel. System.Security.Authentication.AuthenticationException--The remote certificate is invalid according to the validation procedure.


[java  解密碰到的误导人的错误提示 pad block corrupted](http://blog.sina.com.cn/s/blog_54ef398901014ezp.html)




unable to find valid certification path to requested target


到目录jdk/jre/lib/security/下执行命令
keytool -import -trustcacerts -alias casserver -file CtripRootCertificateAuthority.crt -keystore cacerts 密钥口令:changeit


package com.ctrip.microfinance.qunarcooperation.ws.util;

import java.util.Base64;
import java.util.Random;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

public class AES256Util {
	public static String encrypt(String content, String key) {
		try {
			byte[] cipher = encrypt(key.getBytes(), content.getBytes());
			return new String(Base64.getEncoder().encode(cipher));
		} catch (Exception e) {
			return "";
		}
	}

	public static String decrypt(String content, String key) {
		try {
			byte[] cipherBytes = Base64.getDecoder().decode(content);
			byte[] text = decrypt(key.getBytes(), cipherBytes);
			return new String(text);
		} catch (Exception e) {
			return "";
		}
	}

	static byte[] encrypt(byte[] content, byte[] key) {
		try {
			Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
			byte[] aesKeyBytes = new byte[16]; // 32 bytes for AES-256

			int len = key.length > 16 ? 16 : key.length;
			System.arraycopy(key, 0, aesKeyBytes, 0, len);

			SecretKeySpec keySpec = new SecretKeySpec(aesKeyBytes, "AES");

			byte[] iv = new byte[cipher.getBlockSize()];

			Random ran = new Random();
			ran.nextBytes(iv);
			IvParameterSpec ivSpec = new IvParameterSpec(iv);
			cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);

			byte[] results = cipher.doFinal(content);

			byte[] retBytes = new byte[iv.length + results.length];
			System.arraycopy(iv, 0, retBytes, 0, iv.length);
			System.arraycopy(results, 0, retBytes, iv.length, results.length);

			return retBytes;

		} catch (Exception e) {
			return null;
		}
	}

	static byte[] decrypt(byte[] content, byte[] key) {
		try {
			if (content.length < 16 || content.length % 16 != 0) {
				return null;
			}

			Cipher cp = Cipher.getInstance("AES/CBC/PKCS5Padding");
			byte[] aesKeyBytes = new byte[16]; // 32 bytes for AES-256

			int len = key.length > 16 ? 16 : key.length;
			System.arraycopy(key, 0, aesKeyBytes, 0, len);

			SecretKeySpec keySpec = new SecretKeySpec(aesKeyBytes, "AES");

			byte[] iv = new byte[cp.getBlockSize()];

			System.arraycopy(content, 0, iv, 0, iv.length);

			IvParameterSpec ivSpec = new IvParameterSpec(iv);
			cp.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);

			byte[] cip = new byte[content.length - iv.length];

			System.arraycopy(content, iv.length, cip, 0, cip.length);
			byte[] results = cp.doFinal(cip);

			return results;
		} catch (Exception e) {
			return null;
		}
	}

}

[Android客户端加密Server后台端解密 遇到的问题](http://www.jianshu.com/p/733c6da85422)
