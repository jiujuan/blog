**深入理解golang：map**

## map简介
map这种数据结构，使用的非常广泛。因为其在一般情况下o(1)的读写效率。当然最坏情况也有可能是o(n)。
比如：在redis中它叫字典表dictht，还有的叫HashTable，在java里也有很多关于hash的库。其本质都差不多，功能和实现可能有一些区别。

一个哈希表的实现，重点在：
1.hash函数：一个算法不均匀的hash函数，会导致读写效率为o(n)
2.冲突解决：在完美的hash函数，数据量一大，就有可能有key的冲突
3.扩容：数据多了，为了读写性能，hash表可能会扩容

> go version go1.13.9 windows/amd64

## map的内部数据结构

#### hmap结构体

runtime/map.go 
它主要数据结构：hmap 结构体来表示哈希，源码如下：
```Go
// A header for a Go map.
type hmap struct {
    // Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
    // Make sure this stays in sync with the compiler's definition.
    count     int // # live cells == size of map.  Must be first (used by len() builtin)
    flags       uint8
    B             uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
    noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
    hash0        uint32 // hash seed
    buckets      unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
    oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
    nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
    extra       *mapextra // optional fields
}
```

代码注释写的贼好，值得效仿学习。

- `count`： 表示当前哈希表中的元素数量
- `flags`：标识位。
  iterator     = 1   可能有遍历用buckets
  oldIterator  = 2  可能有遍历用oldbuckets，用于扩容期间
  hashWriting  = 4 标记写，用于并发读写检测
  sameSizeGrow = 8  用于等大小buckets扩容，减少overflow桶
- `B` ：表示当前哈希表中 buckets 数量，因为哈希表中桶的数量都是2的倍数，所以该字段会存储对数，也就是len(buckets) == 2^B
- `noverflow` ：溢出 bucket 的个数
- `buckets`：当前桶，长度为（0-2^B）。它是一个指针，指向桶的数组
- `hash0`：哈希种子，它能为hash函数的结果引入随机性，这个值在创建哈希表时候确定，并在调用哈希函数时作为参数传入
- `oldbuckets`：在哈希扩容时用于保存之前 buckets 的字段，它的大小是当前 buckets 的一半。
- `nevacuate`：迁移进度数，标识小于其的buckets已迁移完毕。progress counter for evacuation (buckets less than this have been evacuated)
- `extra` ：额外记录overflow桶信息，即是记录溢出相关的信息.

上面的 buckets 桶就是指向了 bmap，每一个bmap都能存储8个键值对，当哈希中存储数据过多，单个桶无法装满时会使用 extra.overflow 中桶存储溢出的数据。

为啥要使用2个桶呢？ 一个不好吗？
溢出桶是Go语言在使用C语言实现时就使用的设计，它可以减少扩容的频率，所以就使用到现在。（又是一个提高性能的设计技巧）

>上面就是map的底层数据结构了，每个map结构都有很多个buckets（bmap），每个bmap可以存储8个元素，当bmap中的元素超过8个时候，hmap会使用extra中的overflow来扩展存储key。冲突也是用这个来解决的，即是常用的拉链法解决冲突。

extra 扩展桶的 mapextra 结构体：

```Go
// mapextra holds fields that are not present on all maps.
type mapextra struct {
	// If both key and elem do not contain pointers and are inline, then we mark bucket
	// type as containing no pointers. This avoids scanning such maps.
	// However, bmap.overflow is a pointer. In order to keep overflow buckets
	// alive, we store pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
	// overflow and oldoverflow are only used if key and elem do not contain pointers.
	// overflow contains overflow buckets for hmap.buckets.
	// oldoverflow contains overflow buckets for hmap.oldbuckets.
	// The indirection allows to store a pointer to the slice in hiter.
	overflow    *[]*bmap
	oldoverflow *[]*bmap

	// nextOverflow holds a pointer to a free overflow bucket.
	nextOverflow *bmap
}
```

#### bmap结构体：

在文件 runtime/map.go 中

```Go
//runtime/map.go

// A bucket for a Go map.
type bmap struct {
    // tophash generally contains the top byte of the hash value
    // for each key in this bucket. If tophash[0] < minTopHash,
    // tophash[0] is a bucket evacuation state instead.
    tophash [bucketCnt]uint8
    // Followed by bucketCnt keys and then bucketCnt elems.
    // NOTE: packing all the keys together and then all the elems together makes the
    // code a bit more complicated than alternating key/elem/key/elem/... but it allows
    // us to eliminate padding which would be needed for, e.g., map[int64]int8.
    // Followed by an overflow pointer.
}
```

只有一个字段，很意外 - -！。

看英文注释：
tophash 存储了键的哈希的高8位，通过比较不同键的哈希的高8位可以减少访问键值对的次数以提供性能。（又是一个提高性能的设计技巧）

真的只有一个字段吗？ 其实在编译期间会对这个数据结构重建：cmd/compile/internal/gc.map

```Go
type bmap struct {
     topbits [8]uint8   //tophash
     keys [8]keytype
     values [8]valuetype
     pad uintptr
     overflow uintptr
}
```
上面数据结构的K/V存储，是不是很奇怪？keys 和 values 并没有放在一块，不是一组，而是k放一块，v放一块。为哈？
这样做比K/V相邻的好处，方便内存对齐。
比如map[string]int8, v是int8,放一块就避免需要额外内存对齐。

![acf9885cf1799e1e406a1199615de516.png](en-resource://database/8405:1)

## hash值计算

map怎么计算key的hash值？也就是bmap中的topshah如何计算？
它的计算函数位于 runtime/map.go

```Go
// tophash calculates the tophash value for hash.
func tophash(hash uintptr) uint8 {
    top := uint8(hash >> (sys.PtrSize*8 - 8))
    if top < minTopHash {
        top += minTopHash
    }
    return top
}
```
上面的 `minTopHash` 是一个常量，也在 map.go 文件中

```Go
// runtime/map.go

const (
    // Maximum number of key/elem pairs a bucket can hold.
    bucketCntBits = 3
    bucketCnt     = 1 << bucketCntBits
    ... ...
    
    minTopHash     = 5 // minimum tophash for a normal filled cell.
    
    // flags
    iterator     = 1 // there may be an iterator using buckets
    oldIterator  = 2 // there may be an iterator using oldbuckets
    hashWriting  = 4 // a goroutine is writing to the map
    sameSizeGrow = 8 // the current map growth is to a new map of the same size
    // sentinel bucket ID for iterator checks
    noCheck = 1<<(8*sys.PtrSize) - 1
)
```

## 怎么扩容
哈希表中随着元素的增加，哈希表的性能会逐渐恶化，所以需要更多的桶来保证哈希表的性能，这时候就需要扩容了。

怎么判断是否需要扩容？
1.哈希使用了太多溢出桶；
2.装载因子已经超过 6.5；

根据上面的判断扩容的条件，扩容的方式可以分为2类：

1. sameSizeGrow

过多的overflow使用，使用等大小的buckets重新整理，回收多余的overflow桶，提高map读写效率，减少溢出桶占用。

这里借助hmap.noverflow来判断溢出桶是否过多，
hmap.B<=15 时，判断是溢出桶是否多于桶数1<<hmap.B，
否则只判断溢出桶是否多于 1<<15，
这也就是为啥hmap.noverflow，当其接近1<<15 - 1时为近似值, 只要可以评估是否溢出桶过多不合理就行了。

2. biggerSizeOver

count/size > 6.5 (装载因子 :overLoadFactor）, 避免读写效率降低。
扩容一倍，并渐进的在赋值和删除（mapassign和mapdelete）期间，对每个桶重新分流 x（原来桶区间）和y（扩容后的增加的新桶区间）。这里overLoadFactor （count/size）是评估桶的平均装载数据能力，即map平均每个桶装载多少个k/v。
这个值太大，则桶不够用，会有太多溢出桶；太小，则分配了太多桶，浪费了空间。6.5是测试后对map装载能力最大化的一个的选择。

下面是扩容的一些代码：
```Go
// runtime.map.go

// Like mapaccess, but allocates a slot for the key if it is not present in the map.
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
  ... ...
  
again:
    bucket := hash & bucketMask(h.B)
    if h.growing() {
        growWork(t, h, bucket)
    }
    b := (*bmap)(unsafe.Pointer(uintptr(h.buckets) + bucket*uintptr(t.bucketsize)))
    top := tophash(hash)
    var inserti *uint8
    var insertk unsafe.Pointer
    var elem unsafe.Pointer

... ...

    // 这里判断是否扩容
    //1.装载因子已经超过 6.5；
    //2.哈希使用了太多溢出桶；
    // Did not find mapping for key. Allocate new cell & add entry.
    // If we hit the max load factor or we have too many overflow buckets,
    // and we're not already in the middle of growing, start growing.
    if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
        // 提交扩容，生成新桶，记录旧桶相关。但不开始
        // 具体开始是后续赋值和删除期间渐进进行
        hashGrow(t, h)  // 扩容函数入口
        goto again // Growing the table invalidates everything, so try again
    }
    
    //mapassign 或 mapdelete中 渐进扩容
    if h.growing() {
      growWork(t, h, bucket)
    }
... ...

}

// overLoadFactor reports whether count items placed in 1<<B buckets is over loadFactor.
func overLoadFactor(count int, B uint8) bool {
    return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}

const (
    // Maximum number of key/elem pairs a bucket can hold.
    bucketCntBits = 3
    bucketCnt     = 1 << bucketCntBits

    loadFactorNum = 13
    loadFactorDen = 2

)

```

## 参考
- [柴大的go-notes](https://github.com/cch123/golang-notes/blob/master/map.md)
- [dranveness的go语言设计和实现-map](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/)
- [map实现原理](https://segmentfault.com/a/1190000022118894)