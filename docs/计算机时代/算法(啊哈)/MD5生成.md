# Java-MD5生成

MessageDigest类使用步骤：

1. 初始化MessageDigest对象

   ```java
   MessageDigest m=MessageDigest.getInstance("MD5");
   ```

   

2. 调用update方法传入要进行加密的byte数据

   ```java
   m.update(x.getBytes("UTF8" ));
   ```

   

3. （可选）调用reset方法重置要传入进行加密的数据

   ```java
   byte s[ ]=m.digest();
   ```

   

4. 调用digest方法对传入的数据进行加密

   ```java
   private static String encode(byte bytes[]) {
       int i;
       StringBuffer buf = new StringBuffer();
       for (int offset = 0; offset < bytes.length; offset++) {
           i = bytes[offset];
           if (i < 0)
               i += 256;
           if (i < 16)
               buf.append("0");
           buf.append(Integer.toHexString(i));
       }
   
       return buf.toString();
   }
   ```

   