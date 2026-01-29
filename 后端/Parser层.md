# Parser层  
主要三大块  

## 1.Tokenizer类
** 一个工具类,用于辅助parser类进行sql解析**  
```java
public class Tokenizer {
    private byte[] stat;//待解析的原始字节流,sql
    private int pos;//当前解析到 stat 的位置
    private String currentToken;// 缓存的当前 token
    private boolean flushToken;//是否需要解析下一个 token

    //主要提供了这几种方法
    public String peek(){} // 查看当前token
    public void pop(){} //指针移动,下次peek看到下一个token
    /*
     核心方法,返回下一个token
     原理:
      先判断判断是否是空白->移动指针直到不是空白,然后进行下面处理
      判断是否是符号->直接返回 
      判断是否是字母->移动并通过stringbuilder记录,直到不是字母         
    */
    private String next(){}
}
```
## 2.Parser类  
主要目的是接受byte[]的sql然后把对应的sql类返回,内部使用tokenier先获取第一个token来判断是什么类型的sql  
然后通过swith-case把sql交给对应类的方法来处理,最后返回sql类;
```java
switch(token) {
                case "begin":
                    stat = parseBegin(tokenizer);
                    break;
                case "commit":
                    stat = parseCommit(tokenizer);
                    break;
                case "abort":
                    stat = parseAbort(tokenizer);
                    break;
                case "create":
                    stat = parseCreate(tokenizer);
                    break;
                ..........
                case "show":
                    stat = parseShow(tokenizer);
                    break;
                default:
                    throw Error.InvalidCommandException;
```
## 3.statement  
**里面存放了所有的 sql类  
