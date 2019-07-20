# Exchanger

如果两个线程在运行过程中需要交换彼此的信息，比如一个数据或者使用的空间，就需要用到Exchanger这个类，Exchanger为线程交换信息提供了非常方便的途径，它可以作为两个线程交换对象的同步点，只有当每个线程都在进入 exchange ()方法并给出对象时，才能接受其他线程返回时给出的对象。 每次只能两个线程交换数据，如果有多个线程，也只有两个能交换数据。下面看个通俗的例子：一手交钱一首交货！ 

```java
public class ExchangerTest { 
    public static void main(String[] args) { 
        ExecutorService service = Executors.newCachedThreadPool(); 
        final Exchanger exchanger = new Exchanger(); //定义一个交换对象，用来交换数据

        //开启一个线程执行任务
        service.execute(new Runnable(){ 

            @Override
            public void run() { 
                try {                
                    String data1 = "海洛因"; 
                    System.out.println("线程" + Thread.currentThread().getName() 
                            + "正在把毒品" + data1 + "拿出来");                    
                    Thread.sleep((long)(Math.random()*10000)); 

                  //把要交换的数据传到exchange方法中，然后被阻塞，等待另一个线程与之交换。返回交换后的数据
                    String data2 = (String)exchanger.exchange(data1); 

                    System.out.println("线程" + Thread.currentThread().getName() +  
                    "用海洛因换来了" + data2); 
                }catch(Exception e){     
                } finally {
                    service.shutdown();
                    System.out.println("交易完毕，拿着钱快跑！");
                }
            }    
        }); 

        //开启另一个线程执行任务
        service.execute(new Runnable(){ 

            @Override
            public void run() { 
                try {                
                    String data1 = "300万"; 
                    System.out.println("线程" + Thread.currentThread().getName() +  
                    "正在把" + data1 +"拿出来"); 
                    Thread.sleep((long)(Math.random()*10000));   

                    String data2 = (String)exchanger.exchange(data1); 

                    System.out.println("线程" + Thread.currentThread().getName() +  
                    "用300万弄到了" + data2); 
                }catch(Exception e){     
                } finally {
                    service.shutdown();
                    System.out.println("交易完毕，拿着海洛因快跑！");
                }
            }    
        });         
    } 
}  
```

