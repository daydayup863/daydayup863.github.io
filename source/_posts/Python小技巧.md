---
title: Python小技巧 
date: 2020-08-05
tags: python
categories: python
top: 11
description: 列举一些python使用过程中用到的小技巧.
password: 

---

记录一些工作中用到的python小技巧.

<!-- more -->

# 获取ipv4地址
```
def ipv4_interface():
    ipv4_addresses = []
    nets = psutil.net_if_addrs()
    for adapter in nets:
        if adapter == 'lo':
            continue

        for snic in nets[adapter]:
            if snic.family.name != 'AF_INET':
                continue
            ipv4_addresses.append(snic.address)

    return ipv4_addresses
```

# list中dict按某一列排序
```Python
>>> from operator import itemgetter
>>>
>>> sorted(result, key=itemgetter('b'), reverse=True)
[{'c': 1, 'a': 2, 'b': 5}, {'c': 5, 'a': 3, 'b': 4}, {'c': 1, 'a': 1, 'b': 1}, {'c': 5, 'a': 9, 'b': 0}]
>>> sorted(result, key=itemgetter('b'))
[{'c': 5, 'a': 9, 'b': 0}, {'c': 1, 'a': 1, 'b': 1}, {'c': 5, 'a': 3, 'b': 4}, {'c': 1, 'a': 2, 'b': 5}]
>>> sorted(result, key=itemgetter('b'))
[{'c': 5, 'a': 9, 'b': 0}, {'c': 1, 'a': 1, 'b': 1}, {'c': 5, 'a': 3, 'b': 4}, {'c': 1, 'a': 2, 'b': 5}]
>>>
```

# 两个list生成dict
```Python
>>> list1=["a","b","c","d"]
>>> list2=[1,2,3,4]
>>>
>>> dict(zip(list1, list2))
{'c': 3, 'd': 4, 'a': 1, 'b': 2}
>>>
>>> dict(zip(list2, list1))
{1: 'a', 2: 'b', 3: 'c', 4: 'd'}
>>>
>>> list1=["a","b","c","d"]
>>> list2=[1,2,3,4,5]
>>>
>>> dict(zip(list2, list1))
{1: 'a', 2: 'b', 3: 'c', 4: 'd'}
>>>
>>> dict(zip(list1, list2))
{'c': 3, 'd': 4, 'a': 1, 'b': 2}
>>>

```

# List去重
## 方法1
```Python
>>> list1 = [2, 1, 3, 4, 1]
>>> list(set(list1))
```

## 方法2
```
>>> list1 = [2, 1, 3, 4, 1]
>>> {}.fromkeys(list1).keys()
dict_keys([2, 1, 3, 4])
```
## 方法3
```
>>> list1 = [2, 1, 3, 4, 1]
>>> {}.fromkeys(list1).keys()
dict_keys([2, 1, 3, 4])
>>> sorted(set(list1), key=list1.index)
[2, 1, 3, 4]
>>> sorted(set(list1))
[1, 2, 3, 4]
```

# List删除重复项
```
>>> # 方法一:
>>> list1 = [2, 1, 3, 4, 1]
>>> [item for item in list1 if list1.count(item) == 1]
[2, 3, 4]
>>> # 方法二:
>>> list1 = [2, 1, 3, 4, 1]
>>> list(filter(lambda x:list1.count(x) == 1, list1))
[2, 3, 4]
```

