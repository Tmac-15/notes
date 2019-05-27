## dict

字典 (dict) 是一系列无序元素的组合，其长度大小可变，元素 (一对键 key 和值 value 的配对) 可以任意地删减和改变。

#### 创建
```python
d1 = {'name': 'xx', 'age': 18}
d2 = dict({'name': 'xx', 'age': 18})
d3 = dict([('name', 'xx'), ('age', 18)])
d4 = dict(name='xx', age=18)
```

#### 访问
> 可直接索引键，如果不存在，就会抛出异常，也可以使用get(key, default)函数来进行索引。如果键不存在，调用get()函数可以返回一个默认值
```python
d = {'name': 'xx', 'age': 18}

print(d['name']) => 'xx'

print(d['inexistent'])
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
KeyError: 'inexistent'

d.get('name') => 'xx'

d.get('inexistent', 'default') => 'default'
```

#### 新增
```python
d = {'name': 'xx', 'age': 18}

d['gender'] = 'male'

print(d) => {'name': 'xx', 'age': 18, 'gender': 'male'}
```

#### 更新
```python
d = {'name': 'xx', 'age': 18}

d['name'] = 'hah'

print(d) => {'name': 'hah', 'age': 18}
```

#### 删除
```python
d = {'name': 'xx', 'age': 18}

d.pop('name')

print(d) => {'age': 18}
```

#### 排序
```python
d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# 根据字典键的降序(reverse 默认为 False 即升序)
sorted(d.items(), key=lambda x: x[0], reverse=True) => [('d', 4), ('c', 3), ('b', 2), ('a', 1)]

# 根据字典值的降序
sorted(d.items(), key=lambda x: x[1], reverse=True) => [('d', 4), ('c', 3), ('b', 2), ('a', 1)]
```
> 因为字典本身是无序的，所以排序完以后返回一个列表。列表中的每个元素，是由原字典的键和值组成的元组。

#### 原理
> cpython 解释器中字典内部结构是一张哈希表，所以我们可以只用花O(1)的时间完成对一个元素的查找

老版本Python的哈希表结构如下所示：

![image](https://github.com/hhebo/notes/blob/master/images/dict/dict_hash_old.png)

如果你有很多稀疏的哈希表，则这样的设计结构显然会浪费很多内存空间，为了提高存储空间的利用率，用更紧凑的方式来实现哈希表，可以把索引和哈希值、键、值单独分开，也就是下面这样新的结构:

![image](https://github.com/hhebo/notes/blob/master/images/dict/dict_hash_new.png)

> 插入

每次向字典或集合插入一个元素时，Python会首先计算键的哈希值(hash(key))，再和 mask = PyDicMinSize - 1 做与操作，计算这个元素应该插入哈希表的位置 index = hash(key) & mask。如果哈希表中此位置是空的，那么这个元素就会被插入其中。

而如果此位置已被占用，Python便会比较两个元素的哈希值和键是否相等。
* 若两者都相等，则表明这个元素已经存在，如果值不同，则更新值。
* 若两者中有一个不相等，这种情况我们通常称为哈希冲突 (hash collision)，意思是两个元素的键不相等，但是哈希值相等。这种情况下，Python便会继续寻找表中空余的位置，直到找到位置为止。

> 查找

与插入操作类似，Python 会根据哈希值，找到其应该处于的位置;然后，比较哈希表这个位置中元素的哈希值和键，与需要查找的元素是否相等。如果相等，则直接返回；如果不等，则继续查找，直到找到空位或者抛出异常为止。

> 删除

Python会暂时对这个位置的元素，赋于一个特殊的值，等到重新调整哈希表的大小时，再将其删除。

> 哈希冲突的发生，往往会降低哈希操作的速度。因此，为了保证其高效性，哈希表通常会保证其至少留有 1/3 的剩余空间。随着元素的不停插入，当剩余空间小于 1/3 时，Python 会重新获取更大的内存空间，扩充哈希表。不过，这种情况下，表内所有的元素位置都会被重新排放。哈希冲突和哈希表大小的调整，都会导致速度减缓，但是这种情况发生的次数极少。所以，平均情况下，这仍能保证插入、查找和删除的时间复杂度为O(1)。