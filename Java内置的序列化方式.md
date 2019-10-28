---
title: Java内置的序列化方式
date: 2018-06-19 09:07:14
toc: true
categories: Java核心技术
---

网络数据传输的是一个二进制的字节数组。把对象序列化为二进制字节数组和把二进制字节数组反序列化为对象的时间加起来，时间越少，性能越高。使用JSON 和XML的居多！

先看看String类的源码

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
    //....................
}
```

### Java的内置序列化方式

可以看出String实现了Java的内置序列化接口Serializable，于是接下来利用String类演示一下Java的内置序列化是怎样做到的：

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class Demo {
	public static void main(String[] args) throws IOException, ClassNotFoundException {

		String str = new String("Hello");
		
		// 定义一个字节数组输出流
		ByteArrayOutputStream os = new ByteArrayOutputStream();
		
		// 对象输出流
		ObjectOutputStream out = new ObjectOutputStream(os);

		// 将对象写入到字节数组输出，进行序列化
		out.writeObject(str);
		byte[] strByte = os.toByteArray();

		// 字节数组输入流
		ByteArrayInputStream is = new ByteArrayInputStream(strByte);

		// 执行反序列化，从流中读取对象
		ObjectInputStream in = new ObjectInputStream(is);

		String str2 = (String) in.readObject();
		System.out.println(str2);
	}
}
```

![](https://s2.ax1x.com/2019/05/02/EtXmp4.png)

### 使用Hessian进行序列化

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

import com.caucho.hessian.io.HessianInput;
import com.caucho.hessian.io.HessianOutput;

public class Demo {
	public static void main(String[] args) throws IOException {
		String str = new String("Hello");
		
		// 定义一个字节数组输出流
		ByteArrayOutputStream os = new ByteArrayOutputStream();
		
		//Hessian的序列化输出
		HessianOutput ho =  new HessianOutput(os);
		
		ho.writeObject(str);
		
		byte[] strByte = os.toByteArray();
		
		ByteArrayInputStream is = new ByteArrayInputStream(strByte);
		
		//Hessioan的反序列化读取对象
		HessianInput hi = new HessianInput(is);
		
		String str2 = (String)hi.readObject();
		System.out.println(str2);	
	}
}
```

![](https://s2.ax1x.com/2019/05/02/EtX8AK.png)

