# go-cache

在 `github.com/patrickmn/go-cache` 的基础上进行了一些功能添加
* 1. 可注册key删除时，执行后置函数
* 2. 可注册cache flush时，执行的后置函数
* 3. 添加指定时间间隔和指定时刻flush cache的功能 (为了服务长时间运行不出现oom)
* 4. 将初始化方式改为option函数初始化方式

### Installation

`go get github.com/anyanlong/go-cache`

### Usage

```go
import (
	"fmt"
	"github.com/anyanlong/go-cache"
	"time"
)

func main() {
	// 以下初始化效果为
	c := cache.NewCache(
            cache.WithDefaultExpiration(time.Minute), // key默认过期时间为1分钟
            cache.WithCleanupInterval(5*time.Minute), // key全局清理时间为5分钟
            cache.WithItemsInitInterval(5*time.Minute), // cache清空时间间隔为5分钟
            gocache.WithItemsInitTiming(int64(3*3600)), // cache清空时刻为每日凌晨3点
            cache.WithOnFlush(func(cacheSize int) { // 设置cache清空时 执行的函数
                log.WithField("cacheSize", cacheSize).Infoln("go - cache flush!!") 
            }),
            cache.WithOnEvicted(func(k string, v interface{}) { // 设置key被删除时 执行的函数
                log.WithField("k", k).WithField("v", v).Infoln("go - key deleted!!") 
            }),
    )

	// Set the value of the key "foo" to "bar", with the default expiration time
	c.Set("foo", "bar", cache.DefaultExpiration)

	// Set the value of the key "baz" to 42, with no expiration time
	// (the item won't be removed until it is re-set, or removed using
	// c.Delete("baz")
	c.Set("baz", 42, cache.NoExpiration)

	// Get the string associated with the key "foo" from the cache
	foo, found := c.Get("foo")
	if found {
		fmt.Println(foo)
	}

	// Since Go is statically typed, and cache values can be anything, type
	// assertion is needed when values are being passed to functions that don't
	// take arbitrary types, (i.e. interface{}). The simplest way to do this for
	// values which will only be used once--e.g. for passing to another
	// function--is:
	foo, found := c.Get("foo")
	if found {
		MyFunction(foo.(string))
	}

	// This gets tedious if the value is used several times in the same function.
	// You might do either of the following instead:
	if x, found := c.Get("foo"); found {
		foo := x.(string)
		// ...
	}
	// or
	var foo string
	if x, found := c.Get("foo"); found {
		foo = x.(string)
	}
	// ...
	// foo can then be passed around freely as a string

	// Want performance? Store pointers!
	c.Set("foo", &MyStruct, cache.DefaultExpiration)
	if x, found := c.Get("foo"); found {
		foo := x.(*MyStruct)
			// ...
	}
}
```

### Reference

`godoc` or [http://godoc.org/github.com/patrickmn/go-cache](http://godoc.org/github.com/patrickmn/go-cache)
