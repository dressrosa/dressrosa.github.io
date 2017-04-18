---
title: array转list的注意点
date: 2017-04-18 16:03:42
categories: java
tags: 
	- list
---
String phone = "a,b,c,d";
Arrays.asList(phone.split(",")); 

这个产生的是一个固定大小,不可变的数组 ,
是无法进行增加,删除操作的,作用有限 ,jdk里面解释为:
<!--more-->
```
@SafeVarargs
Returns a fixed-size list backed by the specified array. (Changes to the returned list "write through" to the array.) This method acts as bridge between array-based and collection-based APIs, in combination with Collection.toArray. The returned list is serializable and implements RandomAccess.
This method also provides a convenient way to create a fixed-size list initialized to contain several elements:
     List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
Parameters:
a the array by which the list will be backed
Returns:
a list view of the specified array //只是作为一种view形式
```
  
所以要进行操作要用这种:


```
Lists.newArrayList(phone.split(","));
```


```
@GwtCompatible(serializable=true)
Creates a mutableArrayList instance containing the given elements.
Note: if mutability is not required and the elements are non-null, use an overload of ImmutableList.of() (for varargs) or ImmutableList.copyOf(Object[]) (for an array) instead.
Parameters:
elements the elements that the list should contain, in order
Returns:
a new ArrayList containing those elements
```
