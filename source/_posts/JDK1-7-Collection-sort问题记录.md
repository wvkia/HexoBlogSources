---
title: JDK1.7 Collection.sort问题记录
date: 2018-05-10 23:51:07
tags: [JDK,JAVA]
---
#### JDK1.7的Collection.sort()一次错误记录
```language
public class Obj implements Comparable<Obj>{
    private int index;

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }

    @Override
    public int compareTo(Obj o) {
        return getIndex() > o.getIndex() ? 1:0;
    }

    public static void main(String[] args) {

        Obj obj1 = new Obj();
        Obj obj2 = new Obj();
        obj1.setIndex(1);
        obj2.setIndex(2);
        List<Obj> list = new ArrayList<>();
        list.add(obj1);
        list.add(obj2);
        for (Obj obj : list) {
            System.out.println(obj.getIndex());
        }
        Collections.sort(list);
        for (Obj obj : list) {
            System.out.println(obj.getIndex());
        }
    }
}
```
上述代码在对象实现了Comparable接口，并且采用倒叙排序，通过1、0控制，在测试方法中如果正确的话应该结果是
```language
1
2
2
1
```
但在Jdk1.7版本上却是什么都没有变化，根据没有进行排序
又使用具体的Comparator实现了一次
```language
Collections.sort(list, new Comparator<Obj>() {
            @Override
            public int compare(Obj o1, Obj o2) {
                return o1.getIndex() > o2.getIndex()?1:0;
            }
        });
```
同样没有排序
查看jdk源码发现，jdk1.7使用的Timsort算法，严格要求返回值必须互为相反数，1或者-1，而之前版本的jdk使用的归并排序，1、0也可以完成排序
所以目前有两种解决办法：
1. 使用 减法作为结果返回或者1、-1作为结果
2. 设置一个参数启用```System.setProperty("java.util.Arrays.useLegacyMergeSort", "true");  ```强制使用老的排序算法

在对生产环境进行升级时，这种问题需要特别注意。

```language
友情链接：
http://jxlanxin.iteye.com/blog/1814164
http://blog.51cto.com/jerrysun/608416
https://bbs.csdn.net/topics/390249443
```
