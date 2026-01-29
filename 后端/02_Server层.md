# Server类
1.ServerSocket监听端口,有请求则new 一个HandleSocket,HandleSocker类继承了Runnable接口并重写了run方法  
然后交给线程池来执行任务
```java
ThreadPoolExecutor tpe = new ThreadPoolExecutor(
                10,    // 10:核心线程数: 即使空闲也不会被回收,但是可以执行任务
                20,    // 20最大线程数: 当认为队列满时才会创建新线程,直到达到此数量
                1L,    // 1s,空闲线程存活时间,给核心线程空闲多久会被回收
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(100), //容量为100的优先阻塞队列,存储等待执行的任务
                new ThreadPoolExecutor.CallerRunsPolicy() // CallerRunsPolicy拒绝策略,当线程池和队列都满时，由提交任务的线程自己执行任务
        ); // 注意 corepoll(10)->workqueue(100)-> 利用最大线程池(11-20)
```
2.HandleSocket内部使用Packager来接受客户端的sql,交给Excutor执行
```java
Executor exe = new Executor(tbm);
pkg = packager.receive();//拿到客户端的数据
byte[] sql = pkg.getData();
result = exe.execute(sql);
```
3.Excutor依赖"parser层"的解析器将根据sql判断是什么命令并拿到sql类,然后交给"表管理器"处理
```java
Object stat = Parser.Parse(sql);
byte[] res = null;
if(Show.class.isInstance(stat)) {
    res = tbm.show(xid);
} else if(Create.class.isInstance(stat)) {
    res = tbm.create(xid, (Create)stat);
} else if(Select.class.isInstance(stat)) {
    res = tbm.read(xid, (Select)stat);
} else if(Insert.class.isInstance(stat)) {
    res = tbm.insert(xid, (Insert)stat);
} else if(Delete.class.isInstance(stat)) {
    res = tbm.delete(xid, (Delete)stat);
} else if(Update.class.isInstance(stat)) {
    res = tbm.update(xid, (Update)stat);
}
return res;
```
