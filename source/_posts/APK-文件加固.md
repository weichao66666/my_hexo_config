---
layout: article
title: APK 文件加固
date: 2020-08-11 06:19:02
tags:
categories: 
copyright: true
---

# **加固原理**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/1.png)

源码：
1、[ProtectApp](http://colors.black:10086/0/protect-app "http://colors.black:10086/0/protect-app")
2、[ShellAddProject](http://colors.black:10086/0/ShellAddProject "http://colors.black:10086/0/ShellAddProject")

---

# **APK 文件打包流程**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/2.png)

---

# **创建 APK 文件、壳**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/3.png)

```cmd
gradlew assembleDebug
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/4.png)

---

# **加壳流程**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/5.png)

## **加密 DEX 文件**

### **将 APK 文件解压缩**
```java
public static void unZip(File srcFile, File dstDir) {
	try {
		dstDir.delete();
		ZipFile zipFile = new ZipFile(srcFile);
		Enumeration<? extends ZipEntry> entries = zipFile.entries();
		while (entries.hasMoreElements()) {
			ZipEntry zipEntry = entries.nextElement();
			String name = zipEntry.getName();
			if (name.equals("META-INF/CERT.RSA") || name.equals("META-INF/CERT.SF")
					|| name.equals("META-INF/MANIFEST.MF")) {
				continue;
			}

			if (!zipEntry.isDirectory()) {
				File file = new File(dstDir, name);
				if (!file.getParentFile().exists()) {
					file.getParentFile().mkdirs();
				}
				FileOutputStream fos = new FileOutputStream(file);
				InputStream is = zipFile.getInputStream(zipEntry);
				byte[] buffer = new byte[1024];
				int len;
				while ((len = is.read(buffer)) != -1) {
					fos.write(buffer, 0, len);
				}
				is.close();
				fos.close();
			}
		}
		zipFile.close();
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/6.png)

### **将解压缩出来的 DEX 文件加密**
```java
File[] DEXFiles = dstDir.listFiles(new FilenameFilter() {
	@Override
	public boolean accept(File file, String s) {
		return s.endsWith(".dex");
	}
});
for (File file : DEXFiles) {
	// 读数据
	byte[] buffer = Utils.getBytes(file);
	// 加密
	byte[] encryptBytes = encrypt(buffer);
	// 写数据， 替换原来的数据
	FileOutputStream fos = new FileOutputStream(file);
	fos.write(encryptBytes);
	fos.flush();
	fos.close();
}
```

```java
public static byte[] encrypt(byte[] content) {
	try {
		return encryptCipher.doFinal(content);
	} catch (IllegalBlockSizeException e) {
		e.printStackTrace();
	} catch (BadPaddingException e) {
		e.printStackTrace();
	}
	return null;
}
```

### **将 DEX 文件重命名**
classes.dex -> classes_.dex
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/7.png)

## **获取壳的 DEX 文件**

### **将 AAR 文件解压缩**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/8.png)

### **根据解压缩出来的 JAR 文件使用 dx 命令生成 DEX 文件**
```java
File[] files = dir.listFiles(new FilenameFilter() {
	@Override
	public boolean accept(File file, String s) {
		return s.equals("classes.jar");
	}
});
if (files == null || files.length <= 0) {
	throw new RuntimeException("the aar is invalidate");
}
File classesJARFile = files[0];
File classesDEXFile = new File(classesJARFile.getParentFile(), "classes.dex");
dxCommand(classesDEXFile, classesJARFile);
```

```java
public static void dxCommand(File DEXFile, File JARFile) throws IOException, InterruptedException {
	Runtime runtime = Runtime.getRuntime();
	Process process = runtime
			.exec("cmd.exe /C dx --dex --output=" + DEXFile.getAbsolutePath() + " " + JARFile.getAbsolutePath());
	System.out.println("start dx...");

	BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
	String line;
	while ((line = reader.readLine()) != null) {
		System.out.println(line);
	}
	try {
		process.waitFor();
		System.out.println("waiting...");
	} catch (InterruptedException e) {
		e.printStackTrace();
		throw e;
	}

	if (process.exitValue() != 0) {
		InputStream inputStream = process.getErrorStream();
		int len;
		byte[] buffer = new byte[2048];
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		while ((len = inputStream.read(buffer)) != -1) {
			bos.write(buffer, 0, len);
		}
		System.out.println(new String(bos.toByteArray(), "GBK"));
		throw new RuntimeException("dx failed!");
	}
	System.out.println("finish dx!");
	process.destroy();
}
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/9.png)

## **合成新的 APK 文件**

### **打包**
```java
public static void zip(File srcDir, File dstFile) throws Exception {
	dstFile.delete();
	// 对输出文件做 CRC32 校验
	CheckedOutputStream cos = new CheckedOutputStream(new FileOutputStream(dstFile), new CRC32());

	ZipOutputStream zos = new ZipOutputStream(cos);
	compress(srcDir, zos, "");
	zos.flush();
	zos.close();
}

private static void compress(File srcFile, ZipOutputStream zos, String basePath) throws Exception {
	if (srcFile.isDirectory()) {
		compressDir(srcFile, zos, basePath);
	} else {
		compressFile(srcFile, zos, basePath);
	}
}

private static void compressDir(File dir, ZipOutputStream zos, String basePath) throws Exception {
	File[] files = dir.listFiles();
	if (files.length < 1) {
		ZipEntry entry = new ZipEntry(basePath + dir.getName() + "/");
		zos.putNextEntry(entry);
		zos.closeEntry();
	}
	for (File file : files) {
		compress(file, zos, basePath + dir.getName() + "/");
	}
}

private static void compressFile(File file, ZipOutputStream zos, String basePath) throws Exception {
	String name = basePath + file.getName();
	String[] names = name.split("/");

	StringBuilder builder = new StringBuilder();
	if (names.length > 1) {
		for (int i = 1; i < names.length; i++) {
			builder.append("/");
			builder.append(names[i]);
		}
	} else {
		builder.append("/");
	}

	ZipEntry entry = new ZipEntry(builder.toString().substring(1));
	zos.putNextEntry(entry);
	BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
	int count;
	byte data[] = new byte[1024];
	while ((count = bis.read(data, 0, 1024)) != -1) {
		zos.write(data, 0, count);
	}
	bis.close();
	zos.closeEntry();
}
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/10.png)

### **使用 jarsigner 命令签名**
```java
public static void signature(File unsignedAPKFile, File signedAPKFile) throws InterruptedException, IOException {
	// .keystore
//		String cmd[] = { "cmd.exe", "/C ", "jarsigner", "-sigalg", "MD5withRSA", "-digestalg", "SHA1", "-keystore",
//				"source/apk/debug.keystore", "-storepass", "android", "-keypass",
//				"android", "-signedjar", signedApk.getAbsolutePath(), unsignedApk.getAbsolutePath(),
//				"android" };
	// .jks
	String cmd[] = { "cmd.exe", "/C ", "jarsigner", "-verbose", "-keystore", "source/apk/debug.jks", "-storepass",
			"aaaaaa", "-keypass", "aaaaaa", "-signedjar", signedAPKFile.getAbsolutePath(),
			unsignedAPKFile.getAbsolutePath(), "aaaaaa" };
	Process process = Runtime.getRuntime().exec(cmd);
	System.out.println("start sign...");

	BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
	String line;
	while ((line = reader.readLine()) != null) {
		System.out.println(line);
	}
	try {
		process.waitFor();
		System.out.println("waiting...");
	} catch (InterruptedException e) {
		e.printStackTrace();
		throw e;
	}

	if (process.exitValue() != 0) {
		InputStream inputStream = process.getErrorStream();
		int len;
		byte[] buffer = new byte[2048];
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		while ((len = inputStream.read(buffer)) != -1) {
			bos.write(buffer, 0, len);
		}
		System.out.println(new String(bos.toByteArray(), "GBK"));
		throw new RuntimeException("sign failed!");
	}
	System.out.println("finish sign!");
	process.destroy();
}
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/11.png)

---

# **脱壳流程**

## **将 APK 文件解压缩**
```java
public static void unZip(File srcFile, File dstDir) {
    try {
        dstDir.delete();
        ZipFile zipFile = new ZipFile(srcFile);
        Enumeration<? extends ZipEntry> entries = zipFile.entries();
        while (entries.hasMoreElements()) {
            ZipEntry zipEntry = entries.nextElement();
            String name = zipEntry.getName();
            if (name.equals("META-INF/CERT.RSA") || name.equals("META-INF/CERT.SF")
                    || name.equals("META-INF/MANIFEST.MF")) {
                continue;
            }

            if (!zipEntry.isDirectory()) {
                File file = new File(dstDir, name);
                if (!file.getParentFile().exists()) {
                    file.getParentFile().mkdirs();
                }
                FileOutputStream fos = new FileOutputStream(file);
                InputStream is = zipFile.getInputStream(zipEntry);
                byte[] buffer = new byte[1024];
                int len;
                while ((len = is.read(buffer)) != -1) {
                    fos.write(buffer, 0, len);
                }
                is.close();
                fos.close();
            }
        }
        zipFile.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

## **将解压缩出来的 DEX 文件解密**
```java
try {
    decryptCipher = Cipher.getInstance(ALGORITHM);
    byte[] keyStr = password.getBytes();
    SecretKeySpec key = new SecretKeySpec(keyStr, "AES");
    decryptCipher.init(Cipher.DECRYPT_MODE, key);
} catch (NoSuchAlgorithmException e) {
    e.printStackTrace();
} catch (NoSuchPaddingException e) {
    e.printStackTrace();
} catch (InvalidKeyException e) {
    e.printStackTrace();
}
```

```java
public static byte[] decrypt(byte[] content) {
    try {
        return decryptCipher.doFinal(content);
    } catch (IllegalBlockSizeException e) {
        e.printStackTrace();
    } catch (BadPaddingException e) {
        e.printStackTrace();
    }
    return null;
}
```

## **加载解密后的 DEX 文件（插件化原理）**

---

# **验证**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/APK-%E6%96%87%E4%BB%B6%E5%8A%A0%E5%9B%BA/12.gif)

---