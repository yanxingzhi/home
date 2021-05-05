---
layout: post
title:  "造轮子"
date:   2021-05-01 10:00:00 +0800
categories: "文学"
author: yue.yan
tag: "技术"
---

## 1.线程池
> 1.1阻塞队列实现

```java
import java.util.concurrent.BlockingDeque;
import java.util.concurrent.LinkedBlockingDeque;

/**
 * 线程池
 * 概念：线程池管理着一组运行着的线池，它们等待被提交任务去执行
 */
class ThreadPool {
    private int coreNum;
    private BlockingDeque<Runnable> blockingDeque;

    /**
     * 构造线程池
     * @param coreNum 核心线程数
     */
    public ThreadPool(int coreNum) {
        this.coreNum = coreNum;
        this.blockingDeque = new LinkedBlockingDeque();

        for (int i = 0; i < coreNum; i++) {
            new Thread(() -> {
                while (true) {
                    try {
                        Runnable take = blockingDeque.take();
                        take.run();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }

    public void submit(Runnable runnable) {
        blockingDeque.add(runnable);
    }
}
```

> 1.2数组实现

```java
import java.util.List;
import java.util.Objects;
import java.util.concurrent.CopyOnWriteArrayList;

/**
 * 线程池
 * 概念：线程池管理着一组运行着的线池，它们等待被提交任务去执行
 */
class ThreadPool {
    private int coreNum;
    private List<Runnable> list;

    /**
     * 构造线程池
     * @param coreNum 核心线程数
     */
    public ThreadPool(int coreNum) {
        this.coreNum = coreNum;
        this.list = new CopyOnWriteArrayList<>();

        for (int i = 0; i < coreNum; i++) {
            new Thread(() -> {
                while (true) {
                    try {
                        if (list.isEmpty()) {
                            list.wait();
                        }

                        Runnable runnable = null;
                        synchronized (list) {
                            if (!list.isEmpty()) {
                                runnable = list.remove(0);
                            }
                        }
                        
                        if (Objects.nonNull(runnable)) {
                            runnable.run();
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }

    public void submit(Runnable runnable) {
        synchronized (list) {
            if (list.isEmpty()) {
                list.add(runnable);
                list.notify();
                return;
            } else {
                list.add(runnable);
            }
        }
        
    }
}
```

## 2.连接池
