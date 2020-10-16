---
title: 图片压缩Java工具类
date: 2020-04-03 15:49:57
categories:
  - java
tags:
  - java
  - 工具类
---

图片压缩Java工具类
<!--more-->

## 极致压缩

pom.xml
```xml
<dependency>
	<groupId>net.coobird</groupId>
	<artifactId>thumbnailator</artifactId>
	<version>0.4.8</version>
</dependency>
```

java
```java
public String uploadImg2Oss(MultipartFile file) throws Exception {
	if (file.getSize() > 10 * 1024 * 1024) {
		throw new Exception("上传图片大小不能超过10M！");
	}
	String originalFilename = file.getOriginalFilename();
	System.out.println(originalFilename);
	String substring = originalFilename.substring(originalFilename.lastIndexOf(".")).toLowerCase();
	Random random = new Random();
	String name = random.nextInt(10000) + System.currentTimeMillis() + substring;
	try {
		InputStream inputStream = file.getInputStream();
		// 把图片读入到内存中
		BufferedImage bufImg = ImageIO.read(inputStream);
		// 压缩代码
		//设置初始化的压缩率为1
		float ratio = 1f ;
		//设置压缩后的图片宽度为80
		float width = 80f;
		//获取原图片的宽度、高度
		float ImgWith=bufImg.getWidth();
		float ImgHight=bufImg.getHeight();
		//根据原始图片宽度，与压缩图宽度重新计算压缩率
		if (ImgWith>width) {
			ratio=width/ImgWith;
		}
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		//按压缩率进行压缩
		bufImg = Thumbnails.of(bufImg).scale(ratio).asBufferedImage();
		//先转成jpg格式来压缩,然后在通过OSS来修改成源文件本来的后缀格式
		ImageIO.write(bufImg,"jpg",bos);
		//获取输出流
		inputStream = new ByteArrayInputStream(bos.toByteArray());
		//System.out.println("文件名称="+name);
		//上传OSS
		this.uploadFile2OSS(inputStream, name);
		return name;
	} catch (Exception e) {
		throw new Exception("图片上传失败2");
	}
}
```


## 质量压缩
```java
public String uploadImg2Oss(MultipartFile file) throws Exception {
	if (file.getSize() > 10 * 1024 * 1024) {
		throw new Exception("上传图片大小不能超过10M！");
	}
	String originalFilename = file.getOriginalFilename();
	System.out.println(originalFilename);
	String substring = originalFilename.substring(originalFilename.lastIndexOf(".")).toLowerCase();
	Random random = new Random();
	String name = random.nextInt(10000) + System.currentTimeMillis() + substring;
	try {
		InputStream inputStream = file.getInputStream();
		// 把图片读入到内存中
		BufferedImage bufImg = ImageIO.read(inputStream);
		// 压缩代码
		// 存储图片文件byte数组
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		System.out.println(bufImg.getWidth());
		System.out.println(bufImg.getHeight());
		//防止图片变红
		BufferedImage newBufferedImage = new BufferedImage(bufImg.getWidth()-100, bufImg.getHeight()-100, BufferedImage.TYPE_INT_RGB);
		newBufferedImage.createGraphics().drawImage(bufImg, 0, 0, Color.WHITE, null);
		//先转成jpg格式来压缩,然后在通过OSS来修改成源文件本来的后缀格式
		ImageIO.write(newBufferedImage,"jpg",bos);
		//获取输出流
		inputStream = new ByteArrayInputStream(bos.toByteArray());
		//System.out.println("文件名称="+name);
		//上传OSS
		this.uploadFile2OSS(inputStream, name);
		return name;
	} catch (Exception e) {
		throw new Exception("图片上传失败2");
	}
}
```
