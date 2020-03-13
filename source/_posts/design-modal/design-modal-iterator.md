---
title: 行为型-迭代器模式
date: 2019-11-19 09:19:15
categories:
    - 架构之路
tags:
    - 设计模式
---

## 遍历必备迭代器模式

&emsp;&emsp;说到java中用的最多的模式，可能不是工厂，也不是单例，而是迭代器模式。说到编码，循环是必不可少的，循环遍历的方式有许多种，但是不同的数据接口可能有不同的遍历的方式，像java这样具有多态特性的语言，也许在使用的时候并不是到自己不遍历的是那种数据结构，这样就不能使用特定的特性遍历。这个时候，迭代器的出现屏蔽了各个数据结构之间的差异，可以使用统一的方式来遍历。

<!-- more -->
最常使用的List -> ArrayList, 最常见的遍历方式，数组角标遍历:
``` java
public static void main(String[] args) {
    int size = 3;
    List<Integer> numbers = new ArrayList<>(size);
    number.add(1);
    number.add(2);
    number.add(3);

    for (int i = 0; i < size; i++) {
        System.out.println(number.get(i));
    }
}
```
&emsp;&emsp;在集合框架中存在着许多的集合类型，List、set的各种子类，但是他们都实现的是同一个接口：Collection，而在操作一些中间件：mongodb、redis的时候，或者在进行远程调用（rpc）的时候，接口返回会是一个Collection<T>的类型，这样我们就不能直接用上述方式遍历了。而`Collection`继承了一个接口`Iterable`接口，这个接口使得实现类都需要实现iterator方法，使得所有的不同特性的列表数据结构，都可以使用`iterator()` 来获得一个遍历的手段，并且用法是相同的，使用者在遍历的时候不用关心底层到底是什么数据结构（如果需要相关特性的数据结构，可以用`addAll(Collection c)`这个方法）。
&emsp;&emsp;但是Iterator留给我们的印象就是使用的会特别的少，那是因为她换了一种方式，一种简化的方式提供了出来：
``` java
//1
for(Integer num : numbers) {
    System.out.println(num);
}
//2
numbers.foreach(System.out::println);
```
&emsp;&emsp;相信这样的方式一定是用过的，但是他的本质就是使用了iterator，毕竟代码简化还是大多编码者乐意的，毕竟使用iterator首先需要调用`iterator()`来获得一个对象，然后判断`hasNext()`，然后再`next()`获取下一个列表对象...相比较这样的方式，未免也太麻烦了。的确在很多的时候我们就是简单的遍历，使用增强for循环foreach代码是分厂方便的。但是，如果目的不仅仅是遍历，而是边遍历，边添加或者删除元素的话，就会有问题了；
&emsp;&emsp;再增强for或foreach中遍历的时候删除或添加元素，就会报一个`ConcurrentModificationException`的异常，确切的说是删除之后，获取下一个元素的时候报的异常，简单来说就是Iterator中维护了2个字段modCount（当前列表修改的次数）和expectedModCount（预期的修改次数初始和modCount相等），而在ArrayList这样的实现类中，当添加删除元素的时候，modCount的数值变化了，但是expectedModCount却没有，而再`next()`获取元素的时候会有一步检查这2个字段的值是否相等，不相等就出现这个异常。那么为什么使用iterator的`remove()`方法就没问题了呢？那就是iterator的`remove()`方法多了一步，将expectedModCount的值也修改了，这样再单线程中就没有问题了。至于为什么实现类不做这个步骤呢: <b>在多个线程操作集合的时候，其中某个线程通过Iterator遍历线程的时候，就会出现`ConcurrentModificationException`异常，这是为了实现快速失败机制。</b>都知道这是一个线程不安全的集合，那就不应该在多线程的状态下使用它。
&emsp;&emsp;现在先来认识一下`Iterator`做了什么，结构怎样：
``` java
//迭代器的接口
public interface Iterator<E> {
    /**
     * 是否有下一个元素
     */
    boolean hasNext();

    /**
     * 获取下一个元素
     */
    E next();

    /**
     * 删除 默认直接抛异常 因为是要子类去实现的
     */
    default void remove() {throw new UnSupportOperationException("remove");}

    //还有一个1.8的方法 应该是一个访问者模式 这里不展开
}
```
我们的列表需要自己去实现这3个关键接口，那么List中如何实现的呢，在List中实现是在AbstractList中作为一个内部类来实现的（直接把源码拉过来解释）：
``` java
private class Itr implements Iterator<E> {
        /**
         * 应该访问元素的指针
         */
        int cursor = 0;

        /**
         * 访问的上一个元素指针
         */
        int lastRet = -1;

        /**
         * 这里就是提及的预期修改值和修改值，可以看到expectedModCount是Itr类成员变量，而modCount是外部类的成员变量，在使用Iterator的时候exceptedModCount都是单独的，一旦moCount在其他地方修改，那么就会不一致！
         */
        int expectedModCount = modCount;

        public boolean hasNext() {
            // 返回下一个访问指针是否到最大值
            return cursor != size();
        }

        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                //这里做了这个操作
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        //可以看到操作的时候都有这个检查
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```
那么`add()`呢？这个类的源码我肯定贴全了！！！嗯其实它在下一个内部了类中：`ListIterator`的实现类`ListItr`。这个类还继承了Itr，所以拥有哪些计数的属性，之所以分开，我觉得在遍历列表的时候，为什么会有向列表中添加元素的需求呢？删除可以理解，所以应该分析一下，是不是可以把添加元素的操作放到列表的外面来呢？如果实在不行，再用ListIterator吧；

最后总结一下，迭代器没有什么比jdk中的这些迭代器更加典型的了，天天都在用，也需要了解他，java编码规约里也说明使用Iterator来在循环中添加删除元素，这也是一个错误点，但是好在它会报错，不然使用脚标遍历的时候删除，不同的数据可能不会报错，但是结果却是会随着数据的不同而不同，这才要命呢！最后，写一个小贴士：
``` java
public static void main(String[] args) {
    List<Integer> numbers = new ArrayList<>();
    Iterator it = numbers.iterator();
    while(it.hasNext()) {
        //错误1 it.next() 已经获取了下一个元素会使指针下移，所以多次操作的时候先取出来；
        Integer num = it.next();
        if(num < 0) {
            //错误2 numbers.remove() 会报错
            it.remove(num);
        } 
    }
}
```
