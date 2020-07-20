**深入理解golang：sync.map**

## 疑惑开篇
有了map为什么还要搞个sync.map 呢？它们之间有什么区别？
答：重要的一点是，map并发不是安全的。

>在Go 1.6之前， 内置的map类型是部分goroutine安全的，并发的读没有问题，并发的写可能有问题。自go 1.6之后， 并发地读写map会报错，这在一些知名的开源库中都存在这个问题，所以go 1.9之前的解决方案是额外绑定一个锁，封装成一个新的struct或者单独使用锁都可以。


>go version go1.13.9 windows/amd64

### 测试一波
写一个简单的并发写map的测试代码看看：
testcurmap.go
```Go
package main

import (
	"fmt"
	"time"
)

func main() {
	m := map[string]int{"age": 10}

	go func() {
		i := 0
		for i < 1000 {
			m["age"] = 10
			i++
		}
	}()

	go func() { //19 行
		i := 0
		for i < 1000 {
			m["age"] = 11 //22 行
			i++
		}
	}()

	time.Sleep(time.Second * 3)
	fmt.Println(m)
}
```
多运行几次：go run testcurmap.go
会报错，错误的扼要信息如下：

>fatal error: concurrent map writes
>
>goroutine 7 [running]:
>runtime.throw(0x4d49a3, 0x15)
>       /go/src/runtime/panic.go:774 +0x79 fp=0xc000041f30 sp=0xc000041f00 pc=0x42cf19
>runtime.mapassign_faststr(0x4b4360, 0xc000066330, 0x4d168a, 0x3, 0x0)
>        /go/src/runtime/map_faststr.go:211 +0x41e fp=0xc000041f98 sp=0xc000041f30 pc=0x410f8e
>main.main.func2(0xc000066330)
>        /mygo/src/study/go-practice2/map/curmap/testcurmap.go:22 +0x5c fp=0xc000041fd8 sp=0xc000041f98 pc=0x49ac9c
>runtime.goexit()
>      /go/src/runtime/asm_amd64.s:1357 +0x1 fp=0xc000041fe0 sp=0xc000041fd8 pc=0x455391
>created by main.main
>       /mygo/src/study/go-practice2/map/curmap/testcurmap.go:19 +0xb0
>
>exit status 2

看报错信息是src/runtime/map_faststr.go:211 这个函数runtime.mapassign_faststr，它在runtime/map_faststr.go 中，简要代码如下：

```Go
func mapassign_faststr(t *maptype, h *hmap, s string) unsafe.Pointer {
   ... ...
   
	if h.flags&hashWriting != 0 {
		throw("concurrent map writes")
	}
    
    ... ...
}
```
hashWriting  = 4 // a goroutine is writing to the map goroutine写的一个标识，
这里h.flags与自己进行与运算，判断是否有其他goroutine在操作这个map，不是0说明有其他goroutine操作map，所以报错。

**那咋控制map并发呢，一般有几种方式：**

1. map+Mutex：
给map加一把大锁
2. map+RWMutex
给map加一个读写锁，给锁细分。适合读多写少场景


#### 修改一下程序
加一把读写锁防止并发，修改程序 testcurmap2.go：

```Go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	m := map[string]int{"age": 10}

	var s sync.RWMutex
	go func() {
		i := 0
		for i < 1000 {
			s.Lock()
			m["age"] = 10
			s.Unlock()
			i++
		}
	}()

	go func() {
		i := 0
		for i < 1000 {
			s.Lock()
			m["age"] = 11
			s.Unlock()
			i++
		}
	}()

	time.Sleep(time.Second * 3)
	fmt.Println(m)
}
```
运行结果：
map[age:11]

没有报错了。

就到这里了吗？可以在思考思考，还有其他方法控制并发的方法没？有的，sync.map 登场

**控制并的第三种方式：**

3. sync.Map
官方实现的并发map。
原理是通过分离读写map和原子指令来实现读的近似无锁，并通过延迟更新的方式来保证读的无锁化。一般情况下可以替换上面2种锁。

## sync.Map
先看一个简单的代码 testcurmap3.go

```Go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	smap := sync.Map{}

	smap.Store("age", 10)

	go func() {
		i := 0
		for i < 1000 {
			smap.Store("one", 10)
			i++
		}
	}()

	go func() {
		i := 0
		for i < 1000 {
			smap.Store("one", 11)
			i++
		}
	}()

	time.Sleep(time.Second * 2)
	fmt.Println(smap.Load("one"))
}
```
运行输出：11 true
正常输出，没有报错。

**sync.Map 的主要思想就是读写分离，空间换时间**。

看看 sync.map 优点：
1. 空间换时间：通过冗余的两个数据结构(read、dirty)，实现加锁对性能的影响。
2. 使用只读数据(read)，避免读写冲突。
3. 动态调整，miss次数多了之后，将dirty数据迁移到read中。
4. double-checking。
5. 延迟删除。 删除一个键值只是打标记，只有在迁移dirty数据的时候才清理删除的数据。
6. 优先从read读取、更新、删除，因为对read的读取不需要锁。

### sync.Map 数据结构

#### Map 数据结构
在 src/sync/map.go 中
```Go
type Map struct {
    // 当涉及到脏数据(dirty)操作时候，需要使用这个锁
    mu Mutex
    
    // read是一个只读数据结构，包含一个map结构，
    // 读不需要加锁，只需要通过 atomic 加载最新的指正即可
    read atomic.Value // readOnly
    
    // dirty 包含部分map的键值对，如果操作需要mutex获取锁
    // 最后dirty中的元素会被全部提升到read里的map去
    dirty map[interface{}]*entry
    
    // misses是一个计数器，用于记录read中没有的数据而在dirty中有的数据的数量。
    // 也就是说如果read不包含这个数据，会从dirty中读取，并misses+1
    // 当misses的数量等于dirty的长度，就会将dirty中的数据迁移到read中
    misses int
}
```

#### read的数据结构 readOnly：

```Go
// readOnly is an immutable struct stored atomically in the Map.read field.
type readOnly struct {
    // m包含所有只读数据，不会进行任何的数据增加和删除操作 
    // 但是可以修改entry的指针因为这个不会导致map的元素移动
    m       map[interface{}]*entry
    
    // 标志位，如果为true则表明当前read只读map的数据不完整，dirty map中包含部分数据
    amended bool // true if the dirty map contains some key not in m.
}
```
只读map，对该map的访问不需要加锁，但是这个map也不会增加元素，元素会被先增加到dirty中，然后后续会迁移到read只读map中，通过原子操作所以不需要加锁操作。

#### entry

readOnly.m和Map.dirty存储的值类型是*entry,它包含一个指针p, 指向用户存储的value值，结构如下：
```Go
type entry struct {
    p unsafe.Pointer // *interface{}
}
```

p有三种值：
- nil: entry已被删除了，并且m.dirty为nil
- expunged: entry已被删除了，并且m.dirty不为nil，而且这个entry不存在于m.dirty中
- 其它： entry是一个正常的值

## 查找
根据key来查找 value， 函数为 Load()，源码如下：

```Go
// src/sync/map.go

// Load returns the value stored in the map for a key, or nil if no
// value is present.
// The ok result indicates whether value was found in the map.
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
    // 首先从只读ready的map中查找，这时不需要加锁
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    
    // 如果没有找到，并且read.amended为true，说明dirty中有新数据，从dirty中查找，开始加锁了
    if !ok && read.amended {
        m.mu.Lock() // 加锁
        
       // 又在 readonly 中检查一遍，因为在加锁的时候 dirty 的数据可能已经迁移到了read中
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]
        
        // read 还没有找到，并且dirty中有数据
        if !ok && read.amended {
            e, ok = m.dirty[key] //从 dirty 中查找数据
            
            // 不管m.dirty中存不存在，都将misses + 1
            // missLocked() 中满足条件后就会把m.dirty中数据迁移到m.read中
            m.missLocked()
        }
        m.mu.Unlock()
    }
    if !ok {
        return nil, false
    }
    return e.load()
}
```
从函数可以看出，如果查询的键值正好在m.read中，不需要加锁，直接返回结果，优化了性能。
即使不在read中，经过几次miss后， m.dirty中的数据也会迁移到m.read中，这时又可以从read中查找。
所以对于更新／增加较少，加载存在的key很多的case，性能基本和无锁的map类似。

**missLockerd() 迁移数据：**
```Go
// src/sync/map.go

func (m *Map) missLocked() {
    m.misses++
    if m.misses < len(m.dirty) {//misses次数小于 dirty的长度，就不迁移数据，直接返回
        return
    }
    m.read.Store(readOnly{m: m.dirty}) //开始迁移数据
    m.dirty = nil   //迁移完dirty就赋值为nil
    m.misses = 0  //迁移完 misses归0
}
```

## 新增和更新
方法是 Store()， 更新或者新增一个 entry， 源码如下：

```Go
// src/sync/map.go

// Store sets the value for a key.
func (m *Map) Store(key, value interface{}) {
   // 直接在read中查找值，找到了，就尝试 tryStore() 更新值
    read, _ := m.read.Load().(readOnly)
    if e, ok := read.m[key]; ok && e.tryStore(&value) {
        return
    }
    
    // m.read 中不存在
    m.mu.Lock()
    read, _ = m.read.Load().(readOnly)
    if e, ok := read.m[key]; ok {
        if e.unexpungeLocked() { // 未被标记成删除，前面讲到entry数据结构时，里面的p值有3种。1.nil 2.expunged，这个值含义有点复杂，可以看看前面entry数据结构 3.正常值
            
            m.dirty[key] = e // 加入到dirty里
        }
        e.storeLocked(&value) // 更新值
    } else if e, ok := m.dirty[key]; ok { // 存在于 dirty 中，直接更新
        e.storeLocked(&value)
    } else { // 新的值
        if !read.amended { // m.dirty 中没有新数据，增加到 m.dirty 中
            // We're adding the first new key to the dirty map.
            // Make sure it is allocated and mark the read-only map as incomplete.
            m.dirtyLocked() // 从 m.read中复制未删除的数据
            m.read.Store(readOnly{m: read.m, amended: true}) 
        }
        m.dirty[key] = newEntry(value) //将这个entry加入到m.dirty中
    }
    m.mu.Unlock()
}
```

操作都是先从m.read开始，不满足条件再加锁，然后操作m.dirty。

## 删除
根据key删除一个值：

```Go
// src/sync/map.go

// Delete deletes the value for a key.
func (m *Map) Delete(key interface{}) {
    // 从 m.read 中开始查找
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    
    if !ok && read.amended { // m.read中没有找到，并且可能存在于m.dirty中，加锁查找
        m.mu.Lock() // 加锁
        read, _ = m.read.Load().(readOnly) // 再在m.read中查找一次
        e, ok = read.m[key]
        if !ok && read.amended { //m.read中又没找到，amended标志位true，说明在m.dirty中
            delete(m.dirty, key) // 删除
        }
        m.mu.Unlock()
    }
    if ok { // 在 m.ready 中就直接删除
        e.delete()
    }
}
```

还有更好的方法没？java里面有一个分段锁，保证在操作不同 map 段的时候， 可以并发执行， 操作同段 map 的时候，进行锁的竞争和等待。从而达到线程安全， 且效率大于 synchronized。而不是直接加一把大锁，锁住整个map。

那go里面有木有？有人已经想到了

## concurrent-map
项目地址：[concurrent-map](https://github.com/orcaman/concurrent-map)

中文wiki：[地址](https://github.com/orcaman/concurrent-map/blob/master/README-zh.md)

正如 [这里](http://golang.org/doc/faq#atomic_maps) 和 [这里](http://blog.golang.org/go-maps-in-action) 所描述的, Go语言原生的map类型并不支持并发读写。concurrent-map提供了一种高性能的解决方案:通过对内部map进行分片，降低锁粒度，从而达到最少的锁等待时间(锁冲突)。

在Go 1.9之前，go语言标准库中并没有实现并发map。在Go 1.9中，引入了sync.Map。新的sync.Map与此concurrent-map有几个关键区别。标准库中的sync.Map是专为append-only场景设计的。因此，如果您想将Map用于一个类似内存数据库，那么使用我们的版本可能会受益。你可以在golang repo上读到更多，[这里](https://github.com/golang/go/issues/21035) and [这里](https://stackoverflow.com/questions/11063473/map-with-concurrent-access)
>译注: sync.Map在读多写少性能比较好，否则并发性能很差

有兴趣的可以自己研究下。

## 参考：
- [Go 1.9 sync.Map揭秘](https://colobu.com/2017/07/11/dive-into-sync-Map/)
- [go sync.Map使用和介绍](https://blog.csdn.net/u010230794/article/details/82143179)
- [golang sync.map源码](https://github.com/golang/go/blob/master/src/sync/map.go)
