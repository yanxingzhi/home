---
layout: post
title:  "电商产品后台逻辑全解析"
date:   2021-05-01 10:00:00 +0800
categories: "文学"
author: yue.yan
tag: "技术"
---

## 商品中心
### 商品类目

```java

/**
 * 类目
 */
class CatalogDomain {
    /**
     * 类目编号
     */
    private String catalogNo;
    /**
     * 上一级类目
     */
    private String parentCatalogNo;
    /**
     * 类目级别
     */
    private String catalogLevel;
    /**
     * 类目名称
     */
    private String catalog_name;
}

/**
 * 基础数据类目层（后台类目）,主要面向商家后台
 */
class CatalogPostDomain extends CatalogDomain {
    /**
     * 类目映射编号，仅叶子节点做映射
     * @see CatalogPreDomain#catalogNo
     */
    private String mappingNo;
}

/**
 * 前台展示类目层（前台类目），主要面向用户，也可以商家定制
 */
class CatalogPreDomain extends CatalogDomain {
    /**
     * 商家定制自己的类目
     */
    private String merchantNo;
}

/**
 * 类目服务
 */
class CatalogService {
    /**
     * 类目集
     */
    private List<CatalogDomain> catalogList;

    /**
     * TODO by yue.yan
     * 返回所有类目
     * @return
     */
    public List<CatalogDomain> getSortedCatalog() {
        return null;
    }

    /**
     * TODO by yue.yan
     * 返回商家的所有类目
     * @param merchantNo 商家编号
     * @return
     */
    public List<CatalogDomain> getSortedCatalogByMerchantNo(String merchantNo) {
        return null;
    } 
}
```

### 

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
     * 商家录入商品
     * @param skuNo 系统生成的skuNo
     * @param lockedStock 初始数量为 0
     * @param stock 商家填写的库存数 
     */
    public SkuDomain(String skuNo, int lockedStock, int stock) {
        this.skuNo = skuNo;
        this.lockedStock = lockedStock;
        this.stock = stock;
    }

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

```java
/**
 * 商品库存管理，分配模型
 */
class ActivitySkuDomain extends SkuDomain {
    /**
     * 普通商品
     */
    private SkuDomain skuDomain;

    // 商品信息,以sku维度来做库存的扣减
    private String skuNo;
    // 锁定库存
    private int lockedStock;
    // 实际商品库存
    private int stock;
    
    public ActivitySkuDomain(String skuNo, int lockedStock, int stock) {
        super(skuNo, lockedStock, stock);
    }

    /**
     * 参加活动
     * @param skuNo 商家选择参加活动的skuNo
     * @param stock 商家填写的活动库存数
     * @return 是否参加活动成功
     */
    public boolean joinActivity(String skuNo, int stock) {
        skuDomain.lockStock(stock);
        
        this.stock = stock;
        this.lockedStock = 0;
        this.skuNo = skuNo;
        return true;
    }

    /**
     * 活动结束
     * @param skuNo 商家选择参加活动的skuNo
     * @return 是否参加活动成功
     */
    public boolean endActivity(String skuNo) {
        if (0 == this.stock) {
            return true;
        }

        skuDomain.unlockStock(this.stock);
        return true;
    }
}
```

## 订单管理
