---
date:  2015-08-19 17:37
title: '[PHP]Yii2框架的几个隐蔽的坑'
tags: 
- php
- Yii2
- ActiveRecord
- web
- db
- mysql
categories: 
- 技术
---



摘要：Yii2是一款优秀的通用Web后端框架，结构简单优雅、实用功能丰富、扩展性强、性能搞是他最突出的优点。它优秀的地方你在使用过程中总能轻易的发现，无须赘述。而这些隐蔽的小瑕疵，显得更有必要告诉大家。


## 目录
[TOC]

## 说点闲话

距离上次写博客，已经有三个月了。在动手写之前，总是带着深深的罪恶感。被它折磨许久，终于，还是，动手了。

**值得庆祝的一件事**：最近开始，每天早上8：30起来健身了。有两个视频很好用，只需8分钟，照着做一遍保证你（生）爽（不）到（如）爆（死）。（[8分钟腹肌锻炼第2级-下载](http://pan.baidu.com/s/1eQpU4Lk)，[8分钟胸肌锻炼第2级-下载](http://pan.baidu.com/s/1i3KW4T3)）

**值得反思的一件事**：最近看了《叔本华美学随笔》，改变了我一直以来对阅读的看法。我曾经以为阅读是进步的源动力，却被这本书深深的打脸了。来，先给大家分享一段：

>我们只管所见的外在环境并不像阅读物那样，把某已确定的见解强加给我们的头脑，而只是为我们提供了素材和机会。去思考与我们的头脑能力相称、与当下的情绪相符的事情。所以，太多的阅读会是我们的精神失去弹性，就像把一重物持续压在一条弹簧上面就会是弹簧失去弹性一样；而让自己没有自己思想的最稳妥的办法就是在空闲的每一分钟马上随手拿起一本书。

**思考才是进步的源动力**！

好了，扯淡完毕，步入正题。


## ActiveRecord被莫名写入？

### 准备知识

1. `ActiveRecord`的基本用法。如果不理解，可参考[这里](http://www.yiiframework.com/doc-2.0/guide-db-active-record.html)。

### 代码现场


```php
/**
 * @property integer $id
 * @property string $name
 * @property string $detail
 * @property double $price
 * @property integer $area
 **/
class OcRoom extends ActivieRecord
{
    ...
}

$room = OcRoom::find()      //先取出一个对象。
    ->select(['id'])        //只取出'id'列
    ->where(['id'=>20])
    ->one();
$room->save();              //保存，会发现此行的其它字段都被写成默认值了。
```


### 总结问题

这个例子的问题在于：

1. 我从数据库中取出了一行，也就是代码中的`$room`，但是只取出了`id`字段，而其他字段自然就是默认值。
2. 当我`$room->save()`的时候，那些是默认值的字段也被保存到数据库里去了。what!?
3. 也就是说，当你想节约资源，不取出所有字段的时候，**一定要注意不能保存，否则，很多数据会被莫名修改为默认值。**

### 解决方法

然而，我们有什么解决办法呢？提供几种思路：

1. 自己时刻注意，避免未完全取出的`ActiveRecord`的保存。
2. 修改或继承`ActiveRecord`, 使得，当此对象由`find()`新建，且字段没有完全取出，调用`save()`方法，抛出异常。
3. 修改或继承`ActiveRecord`，使得，当此对象由`find()`新建，且字段没有完全取出，调用`save()`方法时，只保存取出过的字段，其他字段被忽略。


## 你的Transaction生效了吗？

### 代码现场

```php
/**
 * @property integer $id
 * @property string $name
 **/
class OcRoom extends ActiveRecord
{
    public function rules()
    {
        return [['name','string','min'=>2,'max'=>10]];
    }
    ...
}
class OcHouse extends ActiveRecord
{
    public function rules()
    {
        return [['name','string','max'=>10]];
    }
    ...
}

$a = new OcRoom();
$a->name = '';                //name为空字符串，不满足rules()条件。

$b = new OcHouse();
$b->name = '我的房间';         //name合法，可以保存。

$transaction = Yii::$app->db->beginTransaction();
try{
    $a->save();               //name字段不合法，无法验证通过，在validate()阶段已经返回false,不会进行数据库存储的步骤，所以也不会抛出异常。
    $b->save();               //name字段合法，可以正常保存。

    $transaction->commit();   //提交后，发现$a保存失败，而$b保存成功。
}
catch (Exception $e) 
{
    Yii::error($e->getTraceAsString(),__METHOD__);
    $transaction->rollBack();
}
```

### 问题总结

这段代码的问题在于：

1. 大家知道`$transaction`的存在意义是保证整段数据库存储代码要么全成功，要么全失败。
2. 显然，在这个例子中，`transaction`并没有达到我们想要的效果：`$a`因为`validate()`都没过，所以`$transation->commit()`的时候并不会报错。

### 解决方法

在`$transation`块内，所有的`save()`都要判断下返回值，如果为`false`，则直接抛出异常。


## 'Y-m-d'不被识别？

### 代码现场

```php
OcRenterBill extends ActiveRecord
{
    public function rules()
    {
        return [
            ['start_time','date','format'=>'Y-m-d'],
        ];
    }
}

$a = new OcRenterBill();
$a = '2015-09-12';
$a->save();                 //会报错，说格式不对。
```

### 问题总结

如果一开始，Yii框架就报错，这个还不算坑。坑的是我在Mac上开发时，这个可以完全正常的工作，而发布到线上环境（Ubuntu）后，就弹出“属性start_time格式无效”的错误。而参考官方文档，发现这种格式是允许的[官方文档](http://www.yiiframework.com/doc-2.0/yii-validators-datevalidator.html)。

啊啊啊。各种试错，最后发现如果改成`php:Y-m-d`，世界就清净了。所以，如果你遇到这种问题，感激我吧。











