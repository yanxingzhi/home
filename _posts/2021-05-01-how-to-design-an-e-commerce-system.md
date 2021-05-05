---
layout: post
title:  "电商产品后台逻辑全解析"
date:   2021-05-01 10:00:00 +0800
categories: "文学"
author: yue.yan
tag: "技术"
---

## 促销活动
> 促销目的
1. 
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.
>
> 


## 库存管理
> 库存管理最重要的是存库的分配和扣减
> 
> 库存分层：销售层、调度层、管理层

### 库存扣减
> 扣减的关键是锁定库存，使用锁定库存的目的之一是

```java
/**
 * 商品存库管理，扣减模型
 */
class SkuDomain {
    // 商品信息,以sku维度来做库存的扣减
    private String skuNo;
    // 锁定库存
    private int lockedStock;
    // 实际商品库存
    private int stock;

    /**
     * @return 可销售库存
     */
    public int calcSellStock() {
        return stock - lockedStock;
    }

    /**
     * 增加库存
     * @param increase 增加数量
     */
    public void addStock(int increase) {
        stock += increase;
    }

    /**
     * 锁定存库
     * @param lockNum 锁定数量
     */
    public void lockStock(int lockNum) {
        lockedStock += lockNum;
    }

    /**
     * 解锁库存,如取消订单
     * @param unlockNum 解锁数量
     */
    public void unlockStock(int unlockNum) {
        lockedStock -= unlockNum;
    }

    /**
     * 配减库存,如下单支付成功
     * @param reduceNum 配减数量
     */
    public void reduceStock(int reduceNum) {
        stock -= reduceNum;
        lockedStock -= reduceNum;
    }

    /**
     * 返还库存,如退货、换货;相当于增加存库
     * @param reentrantNum 返还数量
     */
    public void reentrantStock(int reentrantNum) {
        stock += reentrantNum;
    }
}
```

### 库存分配 TODO
> 活动库存:如果活动库存归为锁定库存，就要分区普通商品和活动商品，

> 预售库存
