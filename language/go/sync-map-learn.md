

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

## 常用接口

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

### 结构体

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
- `dirty`: 负责读写。当read中读取失败计数misses>Len(dirty)时，触发dirty升级为read；当执行Range时，若dirty存在read中没有的元素，也会触发dirty升级为read；

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
    // 加锁和double check
		m.mu.Lock()
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
    // read中不存在，且dirty中存在read没有的元素
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
  // 不到升级条件，直接返回
	if m.misses < len(m.dirty) {
		return
	}
  // dirty升级为read，dirty变为nil，misses计数为0
	m.read.Store(readOnly{m: m.dirty})
	m.dirty = nil
	m.misses = 0
}
```

#### 写步骤

1. 先读read
2. 存在且expunged不为true，尝试直接写入，写入成功则结束；
3. 存在但expunged是true，加锁double check，设置entry的expunged为nil，再设置值
4. 不存在，但dirty中存在，直接将dirty中的entry存储为新值指针；
5. dirty中也不存在，dirty为空，将read拷贝到dirty，再在dirty添加新元素
6. 释放锁

```go
func (m *Map) Store(key, value interface{}) {
  // 判断key是否在read中，若存在，且entry没有被标记删除，尝试直接写入。写入成功则直接返回
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
    // 修改元素从expunged变为nil
		if e.unexpungeLocked() {
			// 这个元素之前被删除了，这意味着有一个非nil的dirty，这个元素不在里面.
			m.dirty[key] = e
		}
    // 更新read map的元素值
		e.storeLocked(&value)
	} else if e, ok := m.dirty[key]; ok {
    // 此时read map没有该元素，但是dirty map有该元素，并需修改dirty map元素值为最新值
		e.storeLocked(&value)
	} else {
    // read.amended==false,说明dirty map为空，需要将read map 复制一份到dirty map
		if !read.amended {
			// 拷贝read 到 dirty中
			m.dirtyLocked()
      // 设置read.amended==true，标明新dirty map有数据
			m.read.Store(readOnly{m: read.m, amended: true})
		}
    // 设置元素进入dirty map，此时dirty map拥有read map和最新设置的元素
    // 这里为何不同步设置read map，这样可以减少miss情况？
		m.dirty[key] = newEntry(value)
	}
  // 释放锁
	m.mu.Unlock()
}
```

tryStore实现：

```go
func (e *entry) tryStore(i *interface{}) bool {
	for {
		p := atomic.LoadPointer(&e.p)
    // 标识为删除，直接返回
		if p == expunged {
			return false
		}
    // cas尝试写入新元素
		if atomic.CompareAndSwapPointer(&e.p, p, unsafe.Pointer(i)) {
			return true
		}
	}
}
```

dirtyLocked实现：(从read中拷贝数据到dirty)

```go
func (m *Map) dirtyLocked() {
  // 拷贝read到dirty的前提是dirty为nil
	if m.dirty != nil {
		return
	}

  // 拷贝元素
	read, _ := m.read.Load().(readOnly)
	m.dirty = make(map[interface{}]*entry, len(read.m))
	for k, e := range read.m {
		if !e.tryExpungeLocked() {
			m.dirty[k] = e
		}
	}
}
```

LoadOrStore实现：

```go
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool) {
   // 先从read中读，
   read, _ := m.read.Load().(readOnly)
   if e, ok := read.m[key]; ok {
      actual, loaded, ok := e.tryLoadOrStore(value)
      if ok {
         return actual, loaded
      }
   }

	// read中不存在
  // 加锁
   m.mu.Lock()
  // 二次检查
   read, _ = m.read.Load().(readOnly)
   if e, ok := read.m[key]; ok {
     // 在read而不在dirty中，需要设置dirty
      if e.unexpungeLocked() {
         m.dirty[key] = e
      }
      actual, loaded, _ = e.tryLoadOrStore(value)
   } else if e, ok := m.dirty[key]; ok {
     // dirty中存在
      actual, loaded, _ = e.tryLoadOrStore(value)
      m.missLocked()
   } else {
     // read 和 dirty 不一致
      if !read.amended {
         // 拷贝read到dirty中
         m.dirtyLocked()
        // 设置read amended为true，
         m.read.Store(readOnly{m: read.m, amended: true})
      }
      m.dirty[key] = newEntry(value)
      actual, loaded = value, false
   }
  // 释放锁
   m.mu.Unlock()

   return actual, loaded
}
```

tryLoadOrStore实现：

```go
func (e *entry) tryLoadOrStore(i interface{}) (actual interface{}, loaded, ok bool) {
   p := atomic.LoadPointer(&e.p)
  // 元素被标记为已删除
   if p == expunged {
      return nil, false, false
   }
  // 元素存在，读取成功
   if p != nil {
      return *(*interface{})(p), true, true
   }

   // 存储新值
   ic := i
   for {
      if atomic.CompareAndSwapPointer(&e.p, nil, unsafe.Pointer(&ic)) {
         return i, false, true
      }
      p = atomic.LoadPointer(&e.p)
      if p == expunged {
         return nil, false, false
      }
      if p != nil {
         return *(*interface{})(p), true, true
      }
   }
}
```

#### 删除实现

删除步骤：

1. 先查看read是否存在元素
2. 不存在且dirty不空，加锁double check；从dirty中取出元素，删除dirty 中元素，增加引入计数；
3. 删除元素值的对象，释放空间

LoadAndDelete实现：

```go
func (m *Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool) {
   // 查看read是否存在要删元素 
   read, _ := m.read.Load().(readOnly)
   e, ok := read.m[key]
  // 不存在，且dirty不为空
   if !ok && read.amended {
      m.mu.Lock()
     // 加锁后的double check
      read, _ = m.read.Load().(readOnly)
      e, ok = read.m[key]
      if !ok && read.amended {
        // 删除dirty里key元素
         e, ok = m.dirty[key]
         delete(m.dirty, key)
         // 增加引用计数
         m.missLocked()
      }
      m.mu.Unlock()
   }
  // 释放元素空间
   if ok {
      return e.delete()
   }
   return nil, false
}
```

entry的delete实现

```go
func (e *entry) delete() (value interface{}, ok bool) {
   for {
      p := atomic.LoadPointer(&e.p)
     // 如果指针为空或已被标记删除，直接返回
      if p == nil || p == expunged {
         return nil, false
      }
     // 执行CAS操作，将entry置为空
      if atomic.CompareAndSwapPointer(&e.p, p, nil) {
         return *(*interface{})(p), true
      }
   }
}
```

#### 遍历

步骤：

1. 判断是否dirty中存在read中没有的key；若存在，将dirty升级为read，重置dirty和misses；
2. 遍历read；

Range实现：

```go
// 参数为一个接收key，value返回bool的函数
func (m *Map) Range(f func(key, value interface{}) bool) {
   read, _ := m.read.Load().(readOnly)
  // 如果amended为true，表示数据都在dirty中
   if read.amended {
      //加锁后double check
      m.mu.Lock()
      read, _ = m.read.Load().(readOnly)
      if read.amended {
        // 将dirty升级为read
         read = readOnly{m: m.dirty}
         m.read.Store(read)
         m.dirty = nil
         m.misses = 0
      }
      m.mu.Unlock()
   }

  // 遍历read即可
   for k, e := range read.m {
      v, ok := e.load()
      if !ok {
         continue
      }
      if !f(k, v) {
         break
      }
   }
}
```

#### 总结：

1. 不管读写，都先检查read；这是因为读read不需要加锁，可以提高性能；
   1. 读的时候，read存在直接返回；read不存在，判断amended是否为false，false再去dirty读；
   2. 写的时候，read存在且expunged不为true，直接修改read中map值的指针；再对expunged为true，dirty存不存在进行考虑；
   3. 遍历，若amended为true，直接遍历read，若为false，将dirty升级为read，遍历read；
2. 加锁之后，都需要进行double check，是为了避免在上一次读取和加锁之间，数据发生了改变，这样前一次读取的数据就是脏数据；
3. 通过amended标记，可以减少去读dirty的频率；
4. 我们发现，在写的时候，会发生从read 拷贝到 dirty，再在dirty添加新元素；读和Range的时候，会有dirty变为read的情况；**因此，我们在使用场景上要注意，尽量一次性写完数据，后面只读；这样就可以减少拷贝的情况**。

## 应用场景

如代码注释和上面的总结，sync.map主要适用于下面两种情况：

- 一次写，多次读等情况；
- 多个协程读、写和覆盖的元素不相交的情况；