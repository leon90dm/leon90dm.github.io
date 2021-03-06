---
layout:     post
title:      "内存占用优化(Trove, Unsafe, ByteBuffer, data compression)"
subtitle:   "内存占用优化(Trove, Unsafe, ByteBuffer, data compression)"
date:       2016-12-12
author:     "BilboDai"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Java
---

假如你的应用需要从一个辅助的数据源获取数据，这个辅助的数据源比如说是一个csv文件。csv文件有一些列，其中一个就是ID字段。我们要做的就是把csv的数据全部导入到内存中并且提供根据ID字段尽可能快的查询。

数据类似：

```
{id=idnum10, surname=Smith10, address=10 One Way Road, Springfield, NJ, DOB=01/11/1965, names=John Paul 10 Ringo}
```

简单的方案 - map of maps
---

`private final Map<String, Map<String, String>> m_data = new HashMap<String, Map<String, String>>( 1000 );` 外部的map用ID作为Key，内部的Map以个字段名作为Key。

测试中我们创建一百万的数据记录，这个方案需要消耗706M的空间来存储实际82M的数据。

Array based entries
---
从上一个方案来看，对于每一条记录都创建了（1+字段数）个Map.Entry，假设csv中每个字段是必须的并且是有序的。那么其实我们可能将里面的Map替换为一个数组，然后用一个全局的数组来存储下标对应的字段名。

```
private Map<String, String[]> m_data = new HashMap<String, String[]>( 1000 );
    //order in which we have to access fields, in order not to hardcode it. We assume no optional fields.
    private List<String> m_keys = new ArrayList<String>( 5 );
----
m_keys = ["ID","surname","address",...]
m_data = {"101":["101","Smith10","10 One Way Road, Springfield, NJ, DOB=01/11/1965",...]}
```

采用这种方式，就节约了内部Map为每个字段创建一个Map.Entry，内存占用降到了495M，相对前面一种方式是一种极大的优化了。

210M(705M-495M)的节约是因为每一个Map.Entry对象需要消耗28字节，csv文件中�有8个字段，而HashMap默认的loadFactor是0.75。所以光是这些Entry对象的占用就达到了28*8=224M的占用。

Array based entry without ID field
---
前面那种方式，ID字段既作为Map的Key，有在数组中有，我们可以ID字段从数据中省略掉：

```
m_keys = ["ID","surname","address",...]
m_data = {"101":["101","Smith10","10 One Way Road, Springfield, NJ, DOB=01/11/1965",...]}

|
v

m_keys = ["ID","surname","address",...]
m_data = {"101":["Smith10","10 One Way Road, Springfield, NJ, DOB=01/11/1965",...]}
```

这样我们又节约了8M的空间。

Packing field values into UTF-8 encoded byte[]
---
如果再想优化，就需要将String转成字节数据进行存储。

```
private Map<String, byte[][]> m_data = new HashMap<String, byte[][]>( 1000 );
    //order in which we have to access fields, in order not to hardcode it. We assume no optional fields.
    private List<String> m_keys = new ArrayList<String>( 5 );
```

这种方案的内存占用为292M，相对上面的方案，又节约了200M（因为每个String对象的内存占用相对字节数据来得多（String对象头12字节+2个hashcode8字节，正好200M）。

Storing all records in a ByteBuffer with a separate index
---
再优化的方法就是反过来优化 byte[][]，我们知道7个字段就需要7个byte[]的对象头也就是12*7M空间。如果将所有的字节数据存在ByteBuffer中，用ID检索到记录在整个数组中的位置，这样1M的记录就可以相对上一个方案节约84M-12M=71M左右的空间。

```
//map from key to buffer position
    protected Map<String, Integer> m_index = new HashMap<String, Integer>( 1000 );

    protected ByteBuffer m_data = ByteBuffer.allocate( BUFFER_SIZE_STEP );
    //order in which we have to access fields, in order not to hardcode it. We assume no optional fields.
    protected List<String> m_keys = new ArrayList<String>( 5 );

---

m_index = {"idnum10":101,...}
m_data = [0x10,...,0x12,...]
m_keys = ["ID","address",...]
```

从而这种方案的实际内存占用223M。

Using hash codes instead of index keys
---
如果再想优化，那么就需要做点牺牲了，将存储ID的字符串改成其hashcode进行索引。

```
 protected Map<String, Integer> m_index = new HashMap<String, Integer>( 1000 );

 |
 v

private Map<Integer, Integer> m_hashIndex = new HashMap<Integer, Integer>( 1000 );

```

理论上会造成两个ID的hashcode是一样的，而实际上可以通过结合两个独立的32bit哈希函数成一个单独的long值来避免。假设能做到。那么这个方案的内存消耗就变成了188M（12+8+字符数据占用-4）。

Use Trove maps, get rid of ID field in the data storage
---
还没结束，如果上面的方案可行，那么我们可以使用Trove进一步减少内存占用。

```
//map from key to buffer position
    private TObjectIntMap<String> m_index = new TObjectIntHashMap<String>( 1000 );
    private TIntIntMap m_hashIndex = new TIntIntHashMap( 1000, 0.75f, -1, -1 ); //0 is valid offset
```

这样内存占用仅仅需要92M即可。
