---
title: "Android应用安全防护和逆向分析-基础篇5-6"
date: 2018-08-29T20:39:26+08:00
lastmod: 2018-08-29T20:39:26+08:00
draft: false
keywords: ["Android",".so","AndroidManifest","resourec.arsc"]
description: "AndroidManifest.xml和resourec.arsc解析"
tags: [
    "android安全",
    "读书笔记"
]
categories: ["读书笔记","Android安全"]
author: "Vorblock"
---

## 一、 基础篇⑤-⑥

这两章主要描述AndroidManifest.xml和resourec.arsc这两个android文件。内容不是很多，下面是两章的笔记。

### 第五章 AndroidManifest.xml格式解析

![AndroidManifest.xml](http://my-md-1253484710.coscd.myqcloud.com/20180829165213.png)

<center>AndroidManifest.xml文件格式图</center>

##### 头部信息

- 文件魔数：4bytes。

- 文件大小：4bytes。

- Chunk内容 头部相同（ChunkType(4bytes)、ChunkSize(4bytes)）。

  - **Sting Chunk** ：主要用于存放AndroidManifest.xml文件中所有的字符串信息。

    - ChunkType：类型，固定4bytes（0x001C001)。
    - ChunkSize：大小，4bytes。
    - StringCount：字符串的个数 ，4bytes。
    - StyleCount ：样式的个数，4bytes。
    - Unknown ：位置区域。4bytes。
    - StringPoolOffset ：字符串池的偏移值。4bytes。偏移值相对于StringChunk头部的位置。
    - StylePoolOffset : 样式池的偏移值。4bytes。没有Style可忽略。
    - StringOffsets ：每一个字符串的偏移值，大小为StringChunk*4。
    - StyleOffsets：每个样式的偏移值，大小为StyleChunk*4。

    如何读取这个文件？

  - **Resourceld Chunk** ：主要用来存放AndroidManifest 中用到的系统属性值对应的资源ID

    - ChunkType：类型，固定4bytes（0x00080108）。
    - ChunkSize：大小，4bytes。
    - ResourceIds : 内容，大小为Resourceld Chunk大小除以4减去头部的8字节。

    解析？

  - **Start Namespace Chunk**：主要包含了AndroidMaifest文件中的命名空间的内容，android中的xml都是采用Schema格式（两种格式DTD和Schema）的，所有肯定有Prefix和URI。

    - Chunk Type：类型，固定4bytes。（0x00100100)。
    - Chunk Size：大小，4bytes。
    - Line Number ：AndroidMaifest文件中行号，4bytes。
    - Unknown：未知区域,4bytes。
    - Prefix：命名空间的前缀（在字符串中的索引值），eg:`android`。
    - Uri：命名空间的URI（在字符串中的索引值），eg:`http://schemas.android.com/apk/res/android`

  - **Start Tag Chunk**：AndroidMaifest.xml的标签信息，最核心的内容，也是最复杂的内容。

    - Chunk Type：类型，固定4bytes。（0x00100102)。
    - Chunk Size：大小，4bytes。
    - Line Number ：对应AndroidMaifest中的行号，4bytes。
    - Unknown：未知区域,4bytes。
    - Namespace Uri ：命名空间的Uri，4bytes。
    - Name：标签名称（在字符串中的索引值），4bytes。
    - Flags：标签的类型，4bytes。eg：是开始标签还是结束标签？
    - Attributes Counk：便签中包含的属性的个数，4bytes。
    - Class Attribute：标签包含的类属性，4bytes。
    - Attributes ：属性内容，每个属性算是一个Entry，Entry是一个大小5的字节数组[Namespace,URI,Name,ValueString,Data]，大小为”属性个数* 5 *4"个字节。

##### AXMLPrinter工具

##### aapt 工具



### 第六章 resourec.arsc文件格式解析

##### 资源文件id格式

![](http://my-md-1253484710.coscd.myqcloud.com/20180829200453.png)

![](http://my-md-1253484710.coscd.myqcloud.com/20180829200408.png)

<center> resourec.arsc文件格式</center>

##### 数据结构

上图

- **头部信息**

   resourec.arsc文件格式由一系列chunk组成，每一个chunk均包含一个ResChunk_header

  ```java
  public class ResChunkHeader{
      public short type;  //当前chunk的类型
      public short headerSize; //当前chunk的头部大小
      public int size;  //当前chunk的大小
      
      public int getHeaderSize(){
      	return 2+2+4    
      }
      @Override
      public String toString(){
          return "type:"+Utils.bytesToHexString(
          Utils.int2Byte(type))+",headerSize:"+headerSize+",size:"+size;
      }
  }
  
  ```

- **资源索引表的头部信息**

   resourec.arsc的第一个结构，结构描述了Resource.arsc文件的大小和资源包数量：

   ```java
   public class ResTableHeader {
    
   	public ResChunkHeader header;  //就是标准的Chunk头部信息格式
   	public int packageCount;  //被编译的资源包的个数
   	
   	public ResTableHeader(){
   		header = new ResChunkHeader();
   	}
   	
   	public int getHeaderSize(){
   		return header.getHeaderSize() + 4;
   	}
   	
   	@Override
   	public String toString(){
   		return "header:"+header.toString()+"\n" + "packageCount:"+packageCount;
   	}
   	
   }
   ```

   ![](http://my-md-1253484710.coscd.myqcloud.com/20180829204728.png)

- **资源项的值字符串资源池**

  包含了所有在资源包里面定义的资源项的值字符串，结构如下：

  ```java
  public class ResStringPoolHeader {
  	
  	public ResChunkHeader header;  //标准的Chunk头部信息结构
  	public int stringCount;  //字符串的个数
  	public int styleCount;  //字符串样式的个数
  	
  	public final static int SORTED_FLAG = 1;
  	public final static int UTF8_FLAG = (1<<8);
  	
  	public int flags;  //字符串的属性,可取值包括0x000(UTF-16),0x001(字符串经过排序)、0X100(UTF-8)和他们的组合值
  	public int stringsStart;  //字符串内容块相对于其头部的距离
  	public int stylesStart;  //字符串样式块相对于其头部的距离
  	
  	public ResStringPoolHeader(){
  		header = new ResChunkHeader();
  	}
  	
  	public int getHeaderSize(){
  		return header.getHeaderSize() + 4 + 4 + 4 + 4 + 4;
  	}
  	
  	@Override
  	public String toString(){
  		return "header:"+header.toString()+"\n" + "stringCount:"+stringCount+",styleCount:"+styleCount+",flags:"+flags+",stringStart:"+stringsStart+",stylesStart:"+stylesStart;
  	}
  	
  }
  ```

  接着头部的的是两个偏移数组，分别是字符串偏移数组和字符串样式偏移数组。这两个偏移数组的大小分别等于stringCount和styleCount的值，而每一个元素的类型都是无符号整型。整个字符中资源池结构如下。![](http://my-md-1253484710.coscd.myqcloud.com/20180829205108.png)

  字符串资源池中的字符串前两个字节为字符串长度,长度计算方法如下：

  `len = (((hbyte & 0x7F) << 8)) | lbyte;`

  如果字符串编码格式为UTF-8则字符串以0X00作为结束符,UTF-16则以0X0000作为结束符。

- **Package数据块**

  这个数据块记录编译包的元数据，头部信息如下：

  ```java
  public class ResTablePackage {
  	
  	public ResChunkHeader header;  //Chunk的头部信息数据结构
  	public int id;  //包的ID,等于Package Id,一般用户包的值Package Id为0X7F,系统资源包的Package Id为0X01；
  	public char[] name = new char[128]; //包名
  	public int typeStrings;  //类型字符串资源池相对头部的偏移
  	public int lastPublicType;  //最后一个导出的Public类型字符串在类型字符串资源池中的索引，目前这个值设置为类型字符串资源池的元素个数。在解析的过程中没发现他的用途
  	public int keyStrings;  //资源项名称字符串相对头部的偏移
  	public int lastPublicKey; // 最后一个导出的Public资源项名称字符串在资源项名称字符串资源池中的索引，目前这个值设置为资源项名称字符串资源池的元素个数。在解析的过程中没发现他的用途
  	
  	public ResTablePackage(){
  		header = new ResChunkHeader();
  	}
  	
  	@Override
  	public String toString(){
  		return "header:"+header.toString()+"\n"+",id="+id+",name:"+name.toString()+",typeStrings:"+typeStrings+",lastPublicType:"+lastPublicType+",keyStrings:"+keyStrings+",lastPublicKey:"+lastPublicKey;
  	}
   
  }
  ```

  ![](http://my-md-1253484710.coscd.myqcloud.com/20180829205652.png)

- **类型规范数据块**

  用来描述资源项的配置差异性。每一种类型都对应有一个类型规范数据块。

  ```java
  
  public class ResTableTypeSpec {
  	
  	public final static int SPEC_PUBLIC = 0x40000000;
  	
  	public ResChunkHeader header;  //Chunk的头部信息结构
  	public byte id;  //标识资源的Type ID,Type ID是指资源的类型ID。资源的类型有animator、anim、color、drawable、layout、menu、raw、string和xml等等若干种，每一种都会被赋予一个ID。
  	public byte res0;  //保留,始终为0
  	public short res1;  //保留,始终为0
  	public int entryCount;  //等于本类型的资源项个数,指名称相同的资源项的个数。
  	
  	public ResTableTypeSpec(){
  		header = new ResChunkHeader();
  	}
  	
  	@Override
  	public String toString(){
  		return "header:"+header.toString()+",id:"+id+",res0:"+res0+",res1:"+res1+",entryCount:"+entryCount;
  	}
  	
  }
  ```

- **资源类型项数据块**

  描述资源项的具体信息，名称、值、配置等信息

  ```java
  public class ResTableType {
  	
  	public ResChunkHeader header;  //Chunk的头部信息结构
  	
  	public final static int NO_ENTRY = 0xFFFFFFFF;
  	
  	public byte id;  //标识资源的Type ID
  	public byte res0;  //保留,始终为0
  	public short res1;  //保留,始终为0
  	public int entryCount;  //等于本类型的资源项个数,指名称相同的资源项的个数。
  	public int entriesStart;  //等于资源项数据块相对头部的偏移值。
  	
  	public ResTableConfig resConfig;  //指向一个ResTable_config,用来描述配置信息,地区,语言,分辨率等
  	
  	public ResTableType(){
  		header = new ResChunkHeader();
  		resConfig = new ResTableConfig();
  	}
   
  	public int getSize(){
  		return header.getHeaderSize() + 1 + 1 + 2 + 4 + 4;
  	}
  	
  	@Override
  	public String toString(){
  		return "header:"+header.toString()+",id:"+id+",res0:"+res0+",res1:"+res1+",entryCount:"+entryCount+",entriesStart:"+entriesStart;
  	}
   
  }
  ```

  ResTable_type后接着是一个大小为entryCount的uint32_t数组，每一个数组元素都用来描述一个资源项数据块的偏移位置。 紧跟在这个偏移数组后面的是一个大小为entryCount的ResTable_entry数组,每一个数组元素都用来描述一个资源项的具体信息。ResTable_entry的结构如下：

  ```java
  public class ResTableEntry {
  	
  	public final static int FLAG_COMPLEX = 0x0001;
  	public final static int FLAG_PUBLIC = 0x0002;
  	
  	public short size;
  	public short flags;
  	
  	public ResStringPoolRef key;
  	
  	public ResTableEntry(){
  		key = new ResStringPoolRef();
  	}
  	
  	public int getSize(){
  		return 2+2+key.getSize();
  	}
  	
  	@Override
  	public String toString(){
  		return "size:"+size+",flags:"+flags+",key:"+key.toString()+",str:"+ParseResourceUtils.getKeyString(key.index);
  	}
   
  }
  ```

  ResTable_entry根据flags的不同,后面跟随的数据也不相同,如果flags此位为1,则ResTable_entry是ResTable_map_entry,ResTable_map_entry继承自ResTable_entry,其结构如下。

  ```java
  public class ResTableMapEntry extends ResTableEntry{
  	
  	public ResTableRef parent;
  	public int count;
  	
  	public ResTableMapEntry(){
  		parent = new ResTableRef();
  	}
  	
  	@Override
  	public int getSize(){
  		return super.getSize() + parent.getSize() + 4;
  	}
  	
  	@Override
  	public String toString(){
  		return super.toString() + ",parent:"+parent.toString()+",count:"+count;
  	}
   
  }
  ```

  ResTable_map_entry其后跟随则count个ResTable_map类型的数组,ResTable_map的结构如下：

  ```java
  package com.wjdiankong.parseresource.type;
   
  /**
   struct ResTable_map
   {
       //bag资源项ID
       ResTable_ref name;
       //bag资源项值
       Res_value value;
   };
   * @author i
   *
   */
  public class ResTableMap {
  	
  	public ResTableRef name;
  	public ResValue value;
  	
  	public ResTableMap(){
  		name = new ResTableRef();
  		value = new ResValue();
  	}
  	
  	public int getSize(){
  		return name.getSize() + value.getSize();
  	}
  	
  	@Override
  	public String toString(){
  		return name.toString()+",value:"+value.toString();
  	}
   
  }
  ```

  如果flags此位为0,则ResTable_entry其后跟随的是一个Res_value,描述一个普通资源的值,Res_value结构如下。

  ```java
  public class ResValue {
  	
  	//dataType字段使用的常量
  	public final static int TYPE_NULL = 0x00;
  	public final static int TYPE_REFERENCE = 0x01;
  	public final static int TYPE_ATTRIBUTE = 0x02;
  	public final static int TYPE_STRING = 0x03;
  	public final static int TYPE_FLOAT = 0x04;
  	public final static int TYPE_DIMENSION = 0x05;
  	public final static int TYPE_FRACTION = 0x06;
  	public final static int TYPE_FIRST_INT = 0x10;
  	public final static int TYPE_INT_DEC = 0x10;
  	public final static int TYPE_INT_HEX = 0x11;
  	public final static int TYPE_INT_BOOLEAN = 0x12;
  	public final static int TYPE_FIRST_COLOR_INT = 0x1c;
  	public final static int TYPE_INT_COLOR_ARGB8 = 0x1c;
  	public final static int TYPE_INT_COLOR_RGB8 = 0x1d;
  	public final static int TYPE_INT_COLOR_ARGB4 = 0x1e;
  	public final static int TYPE_INT_COLOR_RGB4 = 0x1f;
  	public final static int TYPE_LAST_COLOR_INT = 0x1f;
  	public final static int TYPE_LAST_INT = 0x1f;
  	
  	public static final int
      COMPLEX_UNIT_PX			=0,
      COMPLEX_UNIT_DIP		=1,
      COMPLEX_UNIT_SP			=2,
      COMPLEX_UNIT_PT			=3,
      COMPLEX_UNIT_IN			=4,
      COMPLEX_UNIT_MM			=5,
  	COMPLEX_UNIT_SHIFT		=0,
      COMPLEX_UNIT_MASK		=15,
      COMPLEX_UNIT_FRACTION	=0,
      COMPLEX_UNIT_FRACTION_PARENT=1,
      COMPLEX_RADIX_23p0		=0,
      COMPLEX_RADIX_16p7		=1,
      COMPLEX_RADIX_8p15		=2,
      COMPLEX_RADIX_0p23		=3,
      COMPLEX_RADIX_SHIFT		=4,
      COMPLEX_RADIX_MASK		=3,
      COMPLEX_MANTISSA_SHIFT	=8,
      COMPLEX_MANTISSA_MASK	=0xFFFFFF;
  	
  	
  	public short size;  //ResValue的头部大小
  	public byte res0;  //保留，始终为0
  	public byte dataType;  //数据的类型,可以从上面的枚举类型中获取
  	public int data;  //数据对应的索引
  	
  	public int getSize(){
  		return 2 + 1 + 1 + 4;
  	}
  	
  	public String getTypeStr(){
  		switch(dataType){
  			case TYPE_NULL:
  				return "TYPE_NULL";
  			case TYPE_REFERENCE:
  				return "TYPE_REFERENCE";
  			case TYPE_ATTRIBUTE:
  				return "TYPE_ATTRIBUTE";
  			case TYPE_STRING:
  				return "TYPE_STRING";
  			case TYPE_FLOAT:
  				return "TYPE_FLOAT";
  			case TYPE_DIMENSION:
  				return "TYPE_DIMENSION";
  			case TYPE_FRACTION:
  				return "TYPE_FRACTION";
  			case TYPE_FIRST_INT:
  				return "TYPE_FIRST_INT";
  			case TYPE_INT_HEX:
  				return "TYPE_INT_HEX";
  			case TYPE_INT_BOOLEAN:
  				return "TYPE_INT_BOOLEAN";
  			case TYPE_FIRST_COLOR_INT:
  				return "TYPE_FIRST_COLOR_INT";
  			case TYPE_INT_COLOR_RGB8:
  				return "TYPE_INT_COLOR_RGB8";
  			case TYPE_INT_COLOR_ARGB4:
  				return "TYPE_INT_COLOR_ARGB4";
  			case TYPE_INT_COLOR_RGB4:
  				return "TYPE_INT_COLOR_RGB4";
  		}
  		return "";
  	}
  	
  	/*public String getDataStr(){
  		if(dataType == TYPE_STRING){
  			return ParseResourceUtils.getResString(data);
  		}else if(dataType == TYPE_FIRST_COLOR_INT){
  			return Utils.bytesToHexString(Utils.int2Byte(data));
  		}else if(dataType == TYPE_INT_BOOLEAN){
  			return data==0 ? "false" : "true";
  		}
  		return data+"";
  	}*/
  	
  	public String getDataStr() {
  		if (dataType == TYPE_STRING) {
  			return ParseResourceUtils.getResString(data);
  		}
  		if (dataType == TYPE_ATTRIBUTE) {
  			return String.format("?%s%08X",getPackage(data),data);
  		}
  		if (dataType == TYPE_REFERENCE) {
  			return String.format("@%s%08X",getPackage(data),data);
  		}
  		if (dataType == TYPE_FLOAT) {
  			return String.valueOf(Float.intBitsToFloat(data));
  		}
  		if (dataType == TYPE_INT_HEX) {
  			return String.format("0x%08X",data);
  		}
  		if (dataType == TYPE_INT_BOOLEAN) {
  			return data!=0?"true":"false";
  		}
  		if (dataType == TYPE_DIMENSION) {
  			return Float.toString(complexToFloat(data))+
  				DIMENSION_UNITS[data & COMPLEX_UNIT_MASK];
  		}
  		if (dataType == TYPE_FRACTION) {
  			return Float.toString(complexToFloat(data))+
  				FRACTION_UNITS[data & COMPLEX_UNIT_MASK];
  		}
  		if (dataType >= TYPE_FIRST_COLOR_INT && dataType <= TYPE_LAST_COLOR_INT) {
  			return String.format("#%08X",data);
  		}
  		if (dataType >= TYPE_FIRST_INT && dataType <= TYPE_LAST_INT) {
  			return String.valueOf(data);
  		}
  		return String.format("<0x%X, type 0x%02X>",data, dataType);
  	}
  	
  	private static String getPackage(int id) {
  		if (id>>>24==1) {
  			return "android:";
  		}
  		return "";
  	}
  	
  	public static float complexToFloat(int complex) {
  		return (float)(complex & 0xFFFFFF00)*RADIX_MULTS[(complex>>4) & 3];
  	}
  	
  	private static final float RADIX_MULTS[]={
  		0.00390625F,3.051758E-005F,1.192093E-007F,4.656613E-010F
  	};
  	
  	private static final String DIMENSION_UNITS[]={
  		"px","dip","sp","pt","in","mm","",""
  	};
  	
  	private static final String FRACTION_UNITS[]={
  		"%","%p","","","","","",""
  	};
  	
  	@Override
  	public String toString(){
  		return "size:"+size+",res0:"+res0+",dataType:"+getTypeStr()+",data:"+getDataStr();
  	}
   
  }
  ```

  以上代码来自于书的原作者的博客：https://blog.csdn.net/jiangwei0910410003/article/details/50628894

  博客里还有如何解析操作，留看。

### 总结

这两章讲的还是挺详细的，可以留着备用查阅，这两个文件都能加以混淆来保护应用，所以还是挺重要的。