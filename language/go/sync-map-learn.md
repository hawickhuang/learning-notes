# Go sync.map源码学习

## 背景

Go中常用的数据结构：map，并不支持并发，即不是线程安全的。当我们对它进行并发读写的时候，会触发go map的并发读写检测机制，在运行时直接panic：

```
fatal error: concurrent map read and map write
```

```go
func readWriteMap() {
	m := make(map[int]int)

	var wg sync.WaitGroup

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			for j := 0; j < 100; j++ {
				_ = m[1]
			}
			wg.Done()
		}()
	}

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			for j := 0; j < 100; j++ {
				m[2] = 2
			}
			wg.Done()
		}()
	}
	wg.Wait()
}
// test
func Test_readWriteMap(t *testing.T) {
	readWriteMap()
}
```

为了实现对map的并发读写，最简单的做法是加读写锁(RWMutex)，如下所示

```
func readWriteMap() {
	m := make(map[int]int)

	var mu sync.RWMutex
	var wg sync.WaitGroup

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			for j := 0; j < 100; j++ {
				mu.RLock()
				_ = m[1]
				mu.RUnlock()
			}
			wg.Done()
		}()
	}

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			for j := 0; j < 100; j++ {
				mu.Lock()
				m[2] = 2
				mu.Unlock()
			}
			wg.Done()
		}()
	}
	wg.Wait()
}
```

Golang 1.9之后，官方提供了sync.map用来实现并发安全的map，下面基于golang 1.16源码来学习官方是如何实现并发安全的。

## 使用

先看代码示例：

```go
func syncMapReadWrite() {
	var sm sync.Map
	
  // 添加k-v元素
	sm.Store("key1", "value1")
	sm.Store(1, 1)
  // 读取k对应v
	if v, ok := sm.Load("key1"); ok {
		fmt.Printf("load key1=%s\n", v)
	}
	if v, ok := sm.Load(1); ok {
		fmt.Printf("load 1=%d\n", v)
	}

  // 若k存在，则直接读取；否则，添加k-v，并返回v
	if v, ok := sm.LoadOrStore("key1", "value_1"); ok {
		fmt.Printf("LoadOrStore key1=%s\n", v)
	}
	if v, ok := sm.LoadOrStore("2", "2_2"); ok {
		fmt.Printf("LoadOrStore 2=%d\n", v)
	}

  // 遍历map中k-v，需提供func(key, value interface{}) bool 函数
	sm.Range(func(key, value interface{}) bool {
		fmt.Printf("key[%v]=value[%v]\n", key, value)
		return true
	})
}
```

并发读取版本：

```go
func syncMapReadWrite() {
	var sm sync.Map
	var wg sync.WaitGroup

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			sm.Store("key1", "value1")
			sm.Store(1, 1)
		}()
	}
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			if v, ok := sm.Load("key1"); ok {
				fmt.Printf("load key1=%s\n", v)
			}
			if v, ok := sm.Load(1); ok {
				fmt.Printf("load 1=%d\n", v)
			}
		}()
	}

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			if v, ok := sm.LoadOrStore("key1", "value_1"); ok {
				fmt.Printf("LoadOrStore key1=%s\n", v)
			}
			if v, ok := sm.LoadOrStore("2", "2_2"); ok {
				fmt.Printf("LoadOrStore 2=%d\n", v)
			}
		}()
	}

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			sm.Range(func(key, value interface{}) bool {
				fmt.Printf("key[%v]=value[%v]\n", key, value)
				return true
			})
		}()
	}
	wg.Wait()
}
// test
func Test_syncMapReadWrite(t *testing.T) {
	syncMapReadWrite()
}
```

测试后发现，并不会出现普通map的panic报错。下表是sync.map的常用接口：

| 需求                             | 接口                                       | 使用方法                                      |
| -------------------------------- | ------------------------------------------ | --------------------------------------------- |
| 创建sync.map                     | 无                                         | 直接声明即可                                  |
| 添加元素                         | Store                                      | Store("key", "value")                         |
| 读取元素                         | Load                                       | Load("key")                                   |
| 读取元素，若不存在，则设置默认值 | LoadOrStore                                | LoadOrStore("key", "default-value")           |
| 读取后删除                       | LoadAndDelete                              | LoadAndDelete("key")                          |
| 删除                             | Delete                                     | Delete("key")                                 |
| 遍历                             | Range(f func(key, value interface{}) bool) | Range(f func(key, value interface{}) bool {}) |

## 实现

sync.map的定义（保留了主要的英文注释，添加了各字段中文解释）：

```go
// The Map type is optimized for two common use cases: (1) when the entry for a given
// key is only ever written once but read many times, as in caches that only grow,
// or (2) when multiple goroutines read, write, and overwrite entries for disjoint
// sets of keys. In these two cases, use of a Map may significantly reduce lock
// contention compared to a Go map paired with a separate Mutex or RWMutex.
//
// The zero Map is empty and ready for use. A Map must not be copied after first use.
type Map struct {
	mu Mutex // 互斥锁，用于锁定 dirty

  read atomic.Value // 优先读，支持原子操作(类型为下面的readOnly)

	// 数据最新的map，允许读写
	dirty map[interface{}]*entry
	// 记录read中读取不到，而dirty读取的到的次数，用于判断是否将dirty复制到read
	misses int
}
```

readOnly的定义：

```go
// readOnly is an immutable struct stored atomically in the Map.read field.
type readOnly struct {
	m       map[interface{}]*entry // read中的数据存储map
	amended bool // 数据在dirty中，而不在read.m中，该值为true；
}
```

entry的定义：（注意dirty和read里的均使用entry指针，即map里只是存储value指针，value域其实只有一份）

```go
type entry struct {
  // p == nil: 被删除
  // p == expunged: 也是被删除；但是该健在read，不在dirty，
  // 其它：存在对象
	p unsafe.Pointer // *interface{}
}
```

### 实现原理

从上面结构体的定义可以看出，golang的sync.map通过两个数据结构（read，dirty）来存储数据，也就可以联想到空间换时间的概念。分离读和写到两个map，再加上互斥量来做一定的控制逻辑，实现并发读写的功能。

下面来看看read和dirty分别承担的角色：

- `read`: 提供并发读，已存元素的原子写；
- `dirty`: 负责读写。

#### 读步骤：

1. 先读 read map；若存在，直接返回；若不存在，加锁读取dirty map，累加misses；
2. 当misses大于dirty map长度，将dirty map拷贝到read map；

```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
  // 从read中读取 key
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
  // 未找到key，且dirty中存在read中不存在的key
	if !ok && read.amended {
    // 加锁
		m.mu.Lock()
		// 二次检查
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
		if !ok && read.amended {
      // 从dirty中获取key
			e, ok = m.dirty[key]
			// 不管在dirty中是否找到，增加misses计数
			m.missLocked()
		}
    // 释放锁
		m.mu.Unlock()
	}
	if !ok {
		return nil, false
	}
  // 返回数据
	return e.load()
}
```

missLocked的实现：(当misses>len(dirty)时，将dirty的指针赋给read)

```go
func (m *Map) missLocked() {
	m.misses++
	if m.misses < len(m.dirty) {
		return
	}
	m.read.Store(readOnly{m: m.dirty})
	m.dirty = nil
	m.misses = 0
}
```

#### 写步骤

1. 

```go
func (m *Map) Store(key, value interface{}) {
  // 判断key是否在read中，若存在，则直接返回
	read, _ := m.read.Load().(readOnly)
	if e, ok := read.m[key]; ok && e.tryStore(&value) {
		return
	}
	// 不存在
  // 加锁
	m.mu.Lock()
  // 二次检查
	read, _ = m.read.Load().(readOnly)
	if e, ok := read.m[key]; ok {
		if e.unexpungeLocked() {
			// The entry was previously expunged, which implies that there is a
			// non-nil dirty map and this entry is not in it.
			m.dirty[key] = e
		}
		e.storeLocked(&value)
	} else if e, ok := m.dirty[key]; ok {
		e.storeLocked(&value)
	} else {
		if !read.amended {
			// We're adding the first new key to the dirty map.
			// Make sure it is allocated and mark the read-only map as incomplete.
			m.dirtyLocked()
			m.read.Store(readOnly{m: read.m, amended: true})
		}
		m.dirty[key] = newEntry(value)
	}
	m.mu.Unlock()
}
```





## 应用场景