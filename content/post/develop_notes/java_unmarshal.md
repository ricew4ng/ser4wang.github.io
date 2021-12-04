---
title: "Java序列化与反序列化原理"
description: 起因还是之前逛到一些网站，发现关键参数都是用base64编码后的序列化数据，所以序列化了解一下。 从java出发，让我们吹起反序列化的号角吧！
date: 2018-08-24T21:14:23+08:00
tags:
    - Java
categories:
    - develop_notes
---

# 基本

java序列化数据，是通过ObjectOutputStream和ObjectInputStream这两个类来实现的，

举个例子:

> 要序列化的对象data1

```
public class data1 implements Serializable {

    private int id;
    private String name;
    private String pwd;
    private String pwd2;

    public int getId(){ return id; }
    public void setId(int id){ this.id = id; }

    public String getName(){ return name; }
    public void setName(String name){ this.name = name; }

    public String getPwd(){ return pwd; }
    public void setPwd(String pwd){ this.pwd = pwd; }

    public String getPwd2(){ return pwd2; }
    public void setPwd2(String pwd2){ this.pwd2 = pwd2; }
}
```

> 序列化操作类SerializeTest

```
public class SerializeTest {
    public void serialize() throws Exception{
        data1 d = new data1();
        d.setId(1036);
        d.setName("data1");
        d.setPwd("pwd1");
        d.setPwd2("pwd2");
        FileOutputStream fos = new FileOutputStream("d:/project/serial/data1");
        ObjectOutputStream oos = new ObjectOutputStream(fos); //创建Object输出流对象
        oos.writeObject(d); //向data1文件中写入序列化数据data1类
        fos.close();
        oos.close();
        System.out.println("序列化完成");
    }
    public data1 deSerialize() throws Exception{
        FileInputStream fis = new FileInputStream("d:/project/serial/data1");
        ObjectInputStream ois = new ObjectInputStream(fis); //创建Object输入流对象
        data1 d = (data1)ois.readObject(); //从data1文件中反序列化出data1类数据
        ois.close();
        fis.close();
        return d;
    }

    public static void main(String[] args) throws Exception{
        SerializeTest s = new SerializeTest();
        s.serialize();
        data1 d = s.deSerialize();
        System.out.println("id:"+d.getId());
        System.out.println("name:"+d.getName());
        System.out.println("pwd:"+d.getPwd());
    }
}
```

执行后会发现 序列化成功，输出文件data1，同时反序列化成功，我们可以从data1文件中反序列化出data1类，能够获取其中的信息。

我们看看data1文件,notepad打开它长这样

![img](http://p6jpvwsnk.bkt.clouddn.com/18-8-24/51128411.jpg)

再按十六进制打开看看，

![img](http://p6jpvwsnk.bkt.clouddn.com/18-8-24/82094575.jpg)

是的，这就是序列化。

有些字符是能看懂的，比如说data1，id，name，这无疑暴露了很多信息，变量键值对都暴露了，所以网站一般不可能就如此把关键信息这样放进去，应该会有加密，但我们现在首先得知道原理！

和我开始的设想一样，反序列化就是要将这串二进制流按某种编码规则编码。 具体怎么编码，就得要读读源码才能搞明白了。

# **序列化源码解析**

1. 初始化ObjectOutputStream

   序列化靠的是ObjectOutputStream，我们先看它初始化的结果，

   > 构造参数

   ```
    public ObjectOutputStream(OutputStream out) throws IOException {
       verifySubclass();
       bout = new BlockDataOutputStream(out);
       handles = new HandleTable(10, (float) 3.00);
       subs = new ReplaceTable(10, (float) 3.00);
       enableOverride = false;
       writeStreamHeader();
       bout.setBlockDataMode(true);
       if (extendedDebugInfo) {
           debugInfoStack = new DebugTraceInfoStack();
       } else {
           debugInfoStack = null;
       }
   }
   ```

   初始化的代码中，bout是底层字节数据容器

   有行writeStreamHeader()，代码如下：

   ```
   protected void writeStreamHeader() throws IOException {
       bout.writeShort(STREAM_MAGIC);
       bout.writeShort(STREAM_VERSION);
   }
   ```

   writeShort是往容器里写两个字节，这里初始化写入了4个字节(一个STREAM_MAGIC ，一个 STREAM_VERSION)

   ```
   /**
    * Magic number that is written to the stream header.
    */
   final static short STREAM_MAGIC = (short)0xaced;
   
   /**
    * Version number that is written to the stream header.
    */
   final static short STREAM_VERSION = 5;
   ```

   即 ac ed 00 05，表示声明使用序列化协议以及说明序列化版本

2. 开始序列化 writeObject()

   ```
   public final void writeObject(Object obj) throws IOException {
       if (enableOverride) {
           writeObjectOverride(obj);
           return;
       }
       try {
           writeObject0(obj, false);
       } catch (IOException ex) {
           if (depth == 0) {
               writeFatalException(ex);
           }
           throw ex;
       }
   }
   ```

   一般会直接调用writeObject0()

   ```
   private void writeObject0(Object obj, boolean unshared)
       throws IOException
   {
       boolean oldMode = bout.setBlockDataMode(false);
       depth++;
       try {
           // handle previously written and non-replaceable objects
           int h;
   
           ...省略代码
   
           if (obj instanceof ObjectStreamClass) {
               writeClassDesc((ObjectStreamClass) obj, unshared);
               return;
           }
   
           // check for replacement object
           Object orig = obj;
           Class<?> cl = obj.getClass();
           ObjectStreamClass desc;
           for (;;) {
               // REMIND: skip this check for strings/arrays?
               Class<?> repCl;
               desc = ObjectStreamClass.lookup(cl, true);
               if (!desc.hasWriteReplaceMethod() ||
                   (obj = desc.invokeWriteReplace(obj)) == null ||
                   (repCl = obj.getClass()) == cl)
               {
                   break;
               }
               cl = repCl;
           }
   
           // remaining cases
           if (obj instanceof String) {
               writeString((String) obj, unshared);
           } else if (cl.isArray()) {
               writeArray(obj, desc, unshared);
           } else if (obj instanceof Enum) {
               writeEnum((Enum<?>) obj, desc, unshared);
           } else if (obj instanceof Serializable) {
               writeOrdinaryObject(obj, desc, unshared);
           } else {
               if (extendedDebugInfo) {
                   throw new NotSerializableException(
                       cl.getName() + "\n" + debugInfoStack.toString());
               } else {
                   throw new NotSerializableException(cl.getName());
               }
           }
       } finally {
           depth--;
           bout.setBlockDataMode(oldMode);
       }
   }
   ```

   后面那些判断，容易看出，根据对象的不同类型，按不同方法写入序列化数据，这里如果对象实现了Serializable接口，就调用writeOrdinaryObject()方法。

   然后发现这个方法还传入了一个desc，这是在此函数之前的一个for(;;)循环里，创建的用来描述该对象类信息的，ObjectStreamClass类。

   > 然后看writeOrdinaryObject()

   ```
   private void writeOrdinaryObject(Object obj,
                                    ObjectStreamClass desc,
                                    boolean unshared)
       throws IOException
   {
       if (extendedDebugInfo) {
           debugInfoStack.push(
               (depth == 1 ? "root " : "") + "object (class \"" +
               obj.getClass().getName() + "\", " + obj.toString() + ")");
       }
       try {
           desc.checkSerialize();
   
           bout.writeByte(TC_OBJECT);
           writeClassDesc(desc, false);
           handles.assign(unshared ? null : obj);
           if (desc.isExternalizable() && !desc.isProxy()) {
               writeExternalData((Externalizable) obj);
           } else {
               writeSerialData(obj, desc);
           }
       } finally {
           if (extendedDebugInfo) {
               debugInfoStack.pop();
           }
       }
   }
   ```

   先是writeByte()，写入了一个字节的TC_OBJECT标志位(十六进制 73)，然后调用writeClassDesc(desc)，把之前生成的该类信息写入，

   > 跟进看writeClassDesc()

   ```
   private void writeClassDesc(ObjectStreamClass desc, boolean unshared)
       throws IOException
   {
       int handle;
       if (desc == null) {
           writeNull();
       } else if (!unshared && (handle = handles.lookup(desc)) != -1) {
           writeHandle(handle);
       } else if (desc.isProxy()) {
           writeProxyDesc(desc, unshared);
       } else {
           writeNonProxyDesc(desc, unshared);
       }
   }
   ```

   isProxy()判断类是否是动态代理类，没了解过动态代理（先mark），这里因为不是动态代理类，所以会调用writeNonProxyDesc(desc)

   > 跟进writeNonProxyDesc(desc)

   ```
   private void writeNonProxyDesc(ObjectStreamClass desc, boolean unshared)
       throws IOException
   {
       bout.writeByte(TC_CLASSDESC);
       handles.assign(unshared ? null : desc);
   
       if (protocol == PROTOCOL_VERSION_1) {
           // do not invoke class descriptor write hook with old protocol
           desc.writeNonProxy(this);
       } else {
           writeClassDescriptor(desc);
       }
   
       Class<?> cl = desc.forClass();
       bout.setBlockDataMode(true);
       if (cl != null && isCustomSubclass()) {
           ReflectUtil.checkPackageAccess(cl);
       }
       annotateClass(cl);
       bout.setBlockDataMode(false);
       bout.writeByte(TC_ENDBLOCKDATA);
   
       writeClassDesc(desc.getSuperDesc(), false);
   }    
   ```

   发现writeByte写入了一个字节的TC_CLASSDESC(16进制 72)

   然后下面一个判断是true进入writeNonProxy()

   > writeNonProxy()

   ```
   void writeNonProxy(ObjectOutputStream out) throws IOException {
       out.writeUTF(name);
       out.writeLong(getSerialVersionUID());
   
       byte flags = 0;
       if (externalizable) {
           flags |= ObjectStreamConstants.SC_EXTERNALIZABLE;
           int protocol = out.getProtocolVersion();
           if (protocol != ObjectStreamConstants.PROTOCOL_VERSION_1) {
               flags |= ObjectStreamConstants.SC_BLOCK_DATA;
           }
       } else if (serializable) {
           flags |= ObjectStreamConstants.SC_SERIALIZABLE;
       }
       if (hasWriteObjectData) {
           flags |= ObjectStreamConstants.SC_WRITE_METHOD;
       }
       if (isEnum) {
           flags |= ObjectStreamConstants.SC_ENUM;
       }
       out.writeByte(flags);
   
       out.writeShort(fields.length);
       for (int i = 0; i < fields.length; i++) {
           ObjectStreamField f = fields[i];
           out.writeByte(f.getTypeCode());
           out.writeUTF(f.getName());
           if (!f.isPrimitive()) {
               out.writeTypeString(f.getTypeString());
           }
       }
   }
   ```

   调用writeUTF()写入了类名，这个writeUTF()函数，在写入十六进制类名前，会先写入两个字节的类名长度，

   然后再调用writeLong，写入序列化UID

   然后下面有个判断，会判断类接口的实现方式，调用writeByte()写入一个字节的标志位。

   > 下面是所有标志位

   ```
   /**
    * Bit mask for ObjectStreamClass flag. Indicates Externalizable data
    * written in Block Data mode.
    * Added for PROTOCOL_VERSION_2.
    *
    * @see #PROTOCOL_VERSION_2
    * @since 1.2
    */
   final static byte SC_BLOCK_DATA = 0x08;
   
   /**
    * Bit mask for ObjectStreamClass flag. Indicates class is Serializable.
    */
   final static byte SC_SERIALIZABLE = 0x02;
   
   /**
    * Bit mask for ObjectStreamClass flag. Indicates class is Externalizable.
    */
   final static byte SC_EXTERNALIZABLE = 0x04;
   
   /**
    * Bit mask for ObjectStreamClass flag. Indicates class is an enum type.
    * @since 1.5
    */
   final static byte SC_ENUM = 0x10;
   ```

   然后调用writeShort写入两个字节的域长度（比如说有3个变量，就写入 00 03 ）

   接下来是一个循环，准备写入这个类的变量名和它对应的变量类型了

   **每轮循环**:

   1) writeByte写入一个字节的变量类型;

   2) writeUTF()写入变量名

   3) 有个判断，判断是不是原始类型，即是不是对象

   不是基本类型的话，就调用writeTypeString()

   这个writeTypeString()，如果是字符串，就会调用writeString()

   而这个writeString()往往是这样写的，字符串长度(不是大小)小于两个字节，就先写入一个字节的TC_STRING(16进制 74)，然后调用writeUTF()，写入一个signature，这好像跟jvm有关，最后一般写的是

   > 74 00 12 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b

   “翻译”过来就是，字符串类型，占18个字节长度，变量名是 Ljava/lang/string;

   而如果说，之前已经写过刚刚这串74 00 12…3b

   那会调用writeHandle()，先写入一个字节的TC_REFERENCE(16进制 71)，然后调用writeInt()写入 007e0000 + handle，这个handle是之前声明过对象的位置，这里我还没弄清除这个位置是怎么定位的，一般是00 01，也就是说 writeHandle()，一般写入类似

   > 71 00 7e 00 XX

   这样5个字节(最后这个00 XX 还不确定，等我再弄明白，一般是 00 01)

   ------

   上面这些结束了，也就是我们写完了writeNonProxy()，现在再次回到writeNonProxyDesc()

   接下来继续调用writeByte()写入一个字节的TC_ENDBLOCKDATA(16进制 78，表示块结束标志位)。

   再调用writeCLassDesc()，参数是desc的父类。

   这里如果父类没有实现序列化接口那就不会写入，否则回到刚才writeNonProxyDesc那一步开始写父类的类信息和变量信息(起始位72，终止位78)，类似于一个递归调用，最后如果没有实现了序列化接口的父类了，就会调用writeNull()，写入一个字节的TC_NULL(16进制 70)，表示没对象了。

   ------

   好了，总之writeClassDesc()这个递归调用完了之后，我们就回到了writeOrdinaryObject()

   接下来调用writeSerialData()，准备写入序列化数据

   > writeSerialData()

   ```
   private void writeSerialData(Object obj, ObjectStreamClass desc)
       throws IOException
   {
       ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
       for (int i = 0; i < slots.length; i++) {
           ObjectStreamClass slotDesc = slots[i].desc;
           if (slotDesc.hasWriteObjectMethod()) {
               PutFieldImpl oldPut = curPut;
               curPut = null;
               SerialCallbackContext oldContext = curContext;
   
               if (extendedDebugInfo) {
                   debugInfoStack.push(
                       "custom writeObject data (class \"" +
                       slotDesc.getName() + "\")");
               }
               try {
                   curContext = new SerialCallbackContext(obj, slotDesc);
                   bout.setBlockDataMode(true);
                   slotDesc.invokeWriteObject(obj, this);
                   bout.setBlockDataMode(false);
                   bout.writeByte(TC_ENDBLOCKDATA);
               } finally {
                   curContext.setUsed();
                   curContext = oldContext;
                   if (extendedDebugInfo) {
                       debugInfoStack.pop();
                   }
               }
   
               curPut = oldPut;
           } else {
               defaultWriteFields(obj, slotDesc);
           }
       }
   }
   ```

   一个循环，上限是类(包括父类)数量

   **每轮循环:**

   调用defaultWriteFields()

   > defaultWriteFields()

   ```
   private void defaultWriteFields(Object obj, ObjectStreamClass desc)
       throws IOException
   {
       Class<?> cl = desc.forClass();
       if (cl != null && obj != null && !cl.isInstance(obj)) {
           throw new ClassCastException();
       }
   
       desc.checkDefaultSerialize();
   
       int primDataSize = desc.getPrimDataSize();
       if (primVals == null || primVals.length < primDataSize) {
           primVals = new byte[primDataSize];
       }
       desc.getPrimFieldValues(obj, primVals);
       bout.write(primVals, 0, primDataSize, false);
   
       ObjectStreamField[] fields = desc.getFields(false);
       Object[] objVals = new Object[desc.getNumObjFields()];
       int numPrimFields = fields.length - objVals.length;
       desc.getObjFieldValues(obj, objVals);
       for (int i = 0; i < objVals.length; i++) {
           if (extendedDebugInfo) {
               debugInfoStack.push(
                   "field (class \"" + desc.getName() + "\", name: \"" +
                   fields[numPrimFields + i].getName() + "\", type: \"" +
                   fields[numPrimFields + i].getType() + "\")");
           }
           try {
               writeObject0(objVals[i],
                            fields[numPrimFields + i].isUnshared());
           } finally {
               if (extendedDebugInfo) {
                   debugInfoStack.pop();
               }
           }
       }
   }
   ```

   先判断是否是基本类型，是的话调用write直接写入序列化数据

   否则，获取该类所有变量数，开始循环

   **每轮:**

   1) 调用writeObject0()写入变量，也就是说，根据变量类型，用相应方法写入。

   **最后循环结束；**

   随着所有变量的写入，先前一次循环也结束，writeSerialData()方法调用完毕，回到了writeOrdinaryObject()，执行结束回到了writeObject0()，又回到了writeObject()。

   以上，就是序列化全过程，可能还有些细节有疏漏，因为我接触java时间不长，很多地方不明白，如有错误请指正！

# **反序列化源码解析**

待续…

# 小总结

刚开始想的是，要了解反序列化，就得要知道序列化原理，因为逛网站看到很多都是java写的，所以想着先读java的序列化源码，机制应该大多类似，但具体实现我总感觉会有不同，这样子就会给反序列化带来一定困难(?)，因为一开始心里其实想的是写一个通用性反序列化工具，感觉挺有意思，但顶着发烧简单弄明白上面的流程也已经花了三四天了。

感觉反序列化应该是容易实现的一个东西，网站如果只单纯序列化传输数据，容易造成信息泄露，所以一般都会在关键参数加密后再序列化吧…但是不反序列化连看到加密参数的机会都没有，对吧？