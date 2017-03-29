title: fastjson反序列化异常类
date: 2017-03-29
tags: ["fastjson"]
categories:
  笔记
---
## 简介 ## 
因为公司本使用fastjson进行json数据序列化与反序列化。
最近正准备实践微服务，可能需要在各服务之间传递异常，顾将异常信息序列化传输在反序列化。
本以为fastjson根据setter进行反序列化相关操作，但是实际情况有些差距，我自定义的一些字段并没有被反序列化。

## 内容 ##
如下，我的异常结构体是：
```
public class BaseException extends FeignException {

    private int code;

    public BaseException(){
        super("", null);
    }

    protected BaseException(int code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

}
```
我接收到的json数据是：
```
{
    "code": 133,
    "message": "ooo",
    "stackTrace": [...],
    "rootCause": {...}
}
```
但是反序列化后我得到的实体却只有stackTrace信息，并没有code和message信息，只有debug追踪代码找原因。
最后找到DefaultJSONParse.java
```
public <T> T parseObject(Type type, Object fieldName) {
        int token = lexer.token();
        if (token == JSONToken.NULL) {
            lexer.nextToken();
            return null;
        }

        if (token == JSONToken.LITERAL_STRING) {
            if (type == byte[].class) {
                byte[] bytes = lexer.bytesValue();
                lexer.nextToken();
                return (T) bytes;
            }

            if (type == char[].class) {
                String strVal = lexer.stringVal();
                lexer.nextToken();
                return (T) strVal.toCharArray();
            }
        }
        // 这里会根据类型获取相关的反序列化类型，Exception Type对应ThrowableDeserializer反序列化类
        ObjectDeserializer derializer = config.getDeserializer(type);

        try {
            return (T) derializer.deserialze(this, type, fieldName);
        } catch (JSONException e) {
            throw e;
        } catch (Throwable e) {
            throw new JSONException(e.getMessage(), e);
        }
    }
```
ThrowableDeserializer在初始化异常类的代码如下：
```
private Throwable createException(String message, Throwable cause, Class<?> exClass) throws Exception {
        Constructor<?> defaultConstructor = null;
        Constructor<?> messageConstructor = null;
        Constructor<?> causeConstructor = null;
        for (Constructor<?> constructor : exClass.getConstructors()) {
        	Class<?>[] types = constructor.getParameterTypes();
            if (types.length == 0) {
                defaultConstructor = constructor;
                continue;
            }

            if (types.length == 1 && types[0] == String.class) {
                messageConstructor = constructor;
                continue;
            }

            if (types.length == 2 && types[0] == String.class && types[1] == Throwable.class) {
                causeConstructor = constructor;
                continue;
            }
        }

        if (causeConstructor != null) {
            return (Throwable) causeConstructor.newInstance(message, cause);
        }

        if (messageConstructor != null) {
            return (Throwable) messageConstructor.newInstance(message);
        }

        if (defaultConstructor != null) {
            return (Throwable) defaultConstructor.newInstance();
        }

        return null;
    }
```
从最下面的三个if判断可以看出，ThrowableDeserializer是根据构造函数来进行初始化实体的且值支持三种类型，最开始我的构造函数只有无参构造满足最后一个，所以最后出来的数据都为空，除了堆栈。
那堆栈为何不为空呢？
因为在ThrowableDeserializer的deserialze()的最后有一段这样的代码
```
if (stackTrace != null) {
            ex.setStackTrace(stackTrace);
        }

```
so...

## 总结 ## 
使用fastjson反序列化Exception类时一定要给出确定的构造函数且与之对应。
