---
title: 使用CountDownLatch进行压力测试
date: 2018-05-08
tags: [压力测试]
categories: [压力测试]
---

#### 导论
今天突然脑洞打开想用CountDownLatch进行压力测试，感觉应该还挺实用的，试试看吧，正好今天在订单生成的接口需要压测，先来玩玩吧。

#### 代码

```
  package com.mxl.order;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;


/**
 *
 * @author MXL 2018年5月8日
 *
 */
public class CountDownLatchTest implements Runnable{

    final AtomicInteger number = new AtomicInteger();  
    volatile boolean bol = false;  
    static Random rm = new Random();
  
    @Override  
    public void run() {
        
        System.out.println(number.getAndIncrement());  
        List<String> list  =new ArrayList<String>();
        synchronized (this) {  
            try {  
            	System.out.println("当前线程ID为："+Thread.currentThread().getId());
                if (!bol) {  
                    System.out.println(bol);  
                    bol = true;  
                    Thread.sleep(3000);
                }  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            String ss = generateBusinessNo();
            list.add(ss);
            String str = "随机数"+ss;
            System.out.println(str);
            System.out.println("并发数量为" +number.intValue()+"时间为:"+System.currentTimeMillis());  
        }  
    }  
  
    public static void main(String[] args) {

        //使用线程池启动CountDownLatch
        ExecutorService pool = Executors.newCachedThreadPool();
        CountDownLatchTest test = new CountDownLatchTest();
        //500并发，实际没这么大
        for (int i=0;i<500;i++) {  
            pool.execute(test);  
        }

    }  

    private static synchronized String getFixLenthString(int strLength) {
	
        Random rm = new Random();
	// 获得随机数
	double pross = (1 + rm.nextDouble()) * Math.pow(10, strLength);
	// 将获得的获得随机数转化为字符串
	String fixLenthString = String.valueOf(pross);
	// 返回固定的长度的随机数
	return fixLenthString.substring(1, strLength + 1);
		
    }
	
    public static String currentDateString(String pattern) {

        if (pattern == null)
            pattern = "yyyyMMddHHmmss";
        return (new SimpleDateFormat(pattern).format(new Date()));

    }

    /**
     * 这里模拟订单生成规则，时间戳+两位随机数
     * 正常订单包含的不止是这些，这里只做模拟
     * @return
     */
    public static synchronized String generateBusinessNo() {
        String currentDateTime = currentDateString("yyyyMMddHHmmssSSS");
        System.out.println(currentDateTime);
        String no =  currentDateTime +(int)(Math.random()*100);
        System.out.println(no);
//		LoggerUtils.debug(OrderNumberUtil.class, "getFixLenthString orderNo="+no);
        return no;
    }

}  
```
这里模拟500并发同时访问订单生成接口，看看结果，生成订单的generateBusinessNo()方法是加过同步锁的，看结果。

#### 结果

由于输出太多，只贴部分有代表性的输出结果

```
当前线程ID为：32
20180508102713458
2018050810271345892
随机数2018050810271345892
并发数量为500时间为:1525746433458
当前线程ID为：44
20180508102713458
2018050810271345855
随机数2018050810271345855
并发数量为500时间为:1525746433458
当前线程ID为：43
20180508102713458
2018050810271345869
随机数2018050810271345869
并发数量为500时间为:1525746433458
当前线程ID为：42
20180508102713458
201805081027134583
随机数201805081027134583
并发数量为500时间为:1525746433458
```

可以看到线程32、44、43、32生成订单的时间是一样的，若是在这个相同的时间里面生成了相同的随机数那么订单号就重复了，当然真实场景肯定不会这样，真实场景会和用户id，手机号，设备编码一起生成订单号，一个用户不可能在同一时间同时访问500次，除非是黑客攻击。

#### 言归正传
这个是我昨天在看订单系统代码是发现的问题，并一起做了压力测试，想想排除工具的模式用CountDownLatch，虽然有些糙，但是貌似效果是达到了。关于CountDownLatch的使用以及和它类似的CyclicBarrier、Semaphore、ConcurrentHashMap和BlockingQueue并发类请自行百度。