---
title: Python lru_cache 使用与源码解读
date: 2025-01-29 16:35:03
tags:
- Python
- LRU
- LRU Cache
---

## 1. 用法说明

`functools.cache`和`functools.lru_cache`都是Python标准库`functools`模块提供的装饰器，用于缓存函数的计算结果，以提高函数的执行效率。

举一个简单的例子：
```python
from functools import lru_cache
import timeit

@lru_cache
def factorial(n):
    return n * factorial(n-1) if n else 1

execution_time1 = timeit.timeit("factorial(64)", globals=globals(), number=10000)
execution_time2 = timeit.timeit("factorial.__wrapped__(64)", globals=globals(), number=10000)

print(f"Execution time1: {execution_time1:.4f} seconds")
print(f"Execution time2: {execution_time2:.4f} seconds")
print(f"Speedup: {execution_time2/execution_time1:.4f} times")
```
其中`__wrapped__` 表示装饰器中原始的函数，也就是没有作用装饰器之前的裸函数。

代码输出如下：
```bash
Execution time1: 0.0004 seconds
Execution time2: 0.0016 seconds
Speedup: 3.5078 times
```
可以看到，通过lru_cache保存factorial函数的中间结果，得到了3.5倍的加速。
通过这里例子，我们可以看到`lru_cache`的使用方式，也是比较简单：
1. import `lru_cache:`: `from functoools import lru_cache`
2. 给函数添加`@lru_cache`装饰器。

通过查看源码，可以看到`lru_cache`函数签名如下：
```python
def lru_cache(maxsize=128, typed=False):
```
其中`maxsize` 参数表示缓存的最多结果数，默认是128。如果计算结果超过128，则遵循Least-recently-used (LRU)原则，将最近使用次数最少的缓存结果替换为当前的结果。如果设置`maxsize=None`，则缓存无上限，但内存占用也可能会增大，使用时多观察。

`typed`参数表示是否按类型缓存不同变量，即使数值一样。例如`typed=True`，那么`f(decimal.Decimal("3.0"))` 和 `f(3.0)`也会分开缓存。

<!--more-->

### 2. 实际使用例子
上面只是一个玩具例子，实际代码中，`lru_cache`用法还是挺多的，这里举一些实际使用例子，来更清晰地理解它的功能。
#### 2.1 get_available_devices
```python
@lru_cache()
def get_available_devices() -> FrozenSet[str]:
    """
    Returns a frozenset of devices available for the current PyTorch installation.
    """
    devices = {"cpu"}  # `cpu` is always supported as a device in PyTorch

    if is_torch_cuda_available():
        devices.add("cuda")

    if is_torch_mps_available():
        devices.add("mps")

    if is_torch_xpu_available():
        devices.add("xpu")

    if is_torch_npu_available():
        devices.add("npu")

    if is_torch_mlu_available():
        devices.add("mlu")

    if is_torch_musa_available():
        devices.add("musa")

    return frozenset(devices)

```
代码地址： <https://github.com/huggingface/transformers/blob/f11f57c92579aa311dbde5267bc0d8d6f2545f7b/src/transformers/utils/__init__.py#L298>
这是获取所有可用 torch devices的代码，通过增加lru_cache进行缓存。

### 2.2 API请求缓存
```python
import requests
from functools import lru_cache

@lru_cache(maxsize=32)
def get_weather(city: str) -> dict:
    url = f"https://api.weather.com/{city}"
    response = requests.get(url)
    return response.json()

# 多次调用相同城市时，直接从缓存读取
print(get_weather("beijing"))  # 真实请求
print(get_weather("beijing"))  # 命中缓存
```

### 2.3 读取配置
```python
from functools import lru_cache
import configparser

@lru_cache(maxsize=1)  # 只需缓存最新配置
def load_config(config_path: str) -> dict:
    config = configparser.ConfigParser()
    config.read(config_path)
    return {section: dict(config[section]) for section in config.sections()}

# 多次读取同一配置文件时，直接返回缓存对象
config = load_config("app.ini")
```

### 2.4 包含参数的资源初始化
```python
from functools import lru_cache
import tensorflow as tf

@lru_cache(maxsize=2)
def load_model(model_name: str) -> tf.keras.Model:
    print(f"Loading {model_name}...")  # 仅首次加载时打印
    return tf.keras.models.load_model(f"models/{model_name}.h5")

# 重复调用时直接返回已加载模型
model1 = load_model("resnet50")  # 真实加载
model2 = load_model("resnet50")  # 命中缓存
```
## 3. lru_cache源码分析
lru_cache源码在CPython源码目录的`Lib/functools.py`中，可以在GitHub上[查看](https://github.com/python/cpython/blob/main/Lib/functools.py#L480)。
下面通过代码截图的方式详细分析源码。
```python
def lru_cache(maxsize=128, typed=False):
	
    if isinstance(maxsize, int):
        # 如果maxsize为负数，则设置maxsize=0，也就是无缓存
        if maxsize < 0:
            maxsize = 0
    elif callable(maxsize) and isinstance(typed, bool):
        # maxsize没有传入，直接传入的是用户定义函数
        user_function, maxsize = maxsize, 128
        # 调用_lru_cache_wrapper创建wrapper，具体实现在底下
        wrapper = _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo)
        wrapper.cache_parameters = lambda : {'maxsize': maxsize, 'typed': typed}
        # 调用update_wrapper来更新wrapper的元数据，使得与user_function一致
        return update_wrapper(wrapper, user_function)
    elif maxsize is not None:
        raise TypeError(
            'Expected first argument to be an integer, a callable, or None')

    def decorating_function(user_function):
        wrapper = _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo)
        wrapper.cache_parameters = lambda : {'maxsize': maxsize, 'typed': typed}
        return update_wrapper(wrapper, user_function)

    return decorating_function

# LRU装饰器具体实现函数
def _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo):
    # Constants shared by all lru cache instances:
    # 每个object()得到的ID都是唯一的
    sentinel = object()          # unique object used to signal cache misses
    make_key = _make_key         # build a key from the function arguments
    PREV, NEXT, KEY, RESULT = 0, 1, 2, 3   # names for the link fields

    cache = {}
    hits = misses = 0
    full = False
    cache_get = cache.get    # bound method to lookup a key or return None
    cache_len = cache.__len__  # get cache size without calling len()
    lock = RLock()           # because linkedlist updates aren't threadsafe
    root = []                # root of the circular doubly linked list
    root[:] = [root, root, None, None]     # initialize by pointing to self

    if maxsize == 0:

        def wrapper(*args, **kwds):
            # maxsize=0，说明无缓存，直接调用用户函数并返回结果
            nonlocal misses
            misses += 1
            result = user_function(*args, **kwds)
            return result

    elif maxsize is None:

        def wrapper(*args, **kwds):
            # 无限缓存情况，不用考虑LRU替换，直接匹配
            nonlocal hits, misses
            # 生成包含args, kwds和typed的唯一的key
            key = make_key(args, kwds, typed)
            result = cache_get(key, sentinel)
            
            if result is not sentinel:
	            # 找到了key，说明已经有缓存了
                hits += 1
                return result
            misses += 1
            result = user_function(*args, **kwds)
            # 将本次结果进行缓存
            cache[key] = result
            return result

    else:

		# 有缓存大小的情况，需要进行LRU替换
        def wrapper(*args, **kwds):
            # Size limited caching that tracks accesses by recency
            nonlocal root, hits, misses, full
            key = make_key(args, kwds, typed)
            with lock:
                link = cache_get(key)
                if link is not None:
                    # 使用双向链表结构体
                    # 在链表中删除命中的节点
                    link_prev, link_next, _key, result = link
                    link_prev[NEXT] = link_next
                    link_next[PREV] = link_prev
				    # 将命中的节点移动到最后位置，root为最开始位置，表示最旧没用的数据，而last表示最新使用的数据
                    last = root[PREV]
                    last[NEXT] = root[PREV] = link
                    link[PREV] = last
                    link[NEXT] = root
                    hits += 1
                    return result
                misses += 1
            result = user_function(*args, **kwds)
	        
	        #处理没命中的情况，因为如果命中的话，前面已经return了
            with lock:
                if key in cache:
                    # 这种情况说明别的线程写入了key，由于节点已经移动到最开始位置了，这里不需要做操作，只需要确保结果最后会返回
                    pass
                elif full:
		            # 缓存已满，需要LRU替换
                    # 使用要删除的节点保存新插入的数据，避免额外的内存申请
                    oldroot = root
                    oldroot[KEY] = key
                    oldroot[RESULT] = result
					
                    root = oldroot[NEXT]
                    oldkey = root[KEY]
                    oldresult = root[RESULT]
                    root[KEY] = root[RESULT] = None
                    # Now update the cache dictionary.
                    del cache[oldkey]
                    # Save the potentially reentrant cache[key] assignment
                    # for last, after the root and links have been put in
                    # a consistent state.
                    cache[key] = oldroot
                else:
                    # 没满的时候，直接插入link到最后
                    last = root[PREV]
                    link = [last, root, key, result]
                    last[NEXT] = root[PREV] = cache[key] = link
                    # 检查缓存是否满了，使用cache_len而不是len()，因为len()可能会被lru_cache缓存,但属性不会，cache_len = cache.__len__
                    full = (cache_len() >= maxsize)
            return result

    def cache_info():
        """Report cache statistics"""
        with lock:
            return _CacheInfo(hits, misses, maxsize, cache_len())

    def cache_clear():
        """Clear the cache and cache statistics"""
        # 清空cache
        nonlocal hits, misses, full
        with lock:
            cache.clear()
            root[:] = [root, root, None, None]
            hits = misses = 0
            full = False

    wrapper.cache_info = cache_info
    wrapper.cache_clear = cache_clear
    return wrapper
```


## 4. lru_cache和cache的区别

`functools.cache`是Python 3.9引入的新特性，作为`lru_cache`的无缓存大小限制的一个alias。
具体来说，通过查看[源码](https://github.com/python/cpython/blob/main/Lib/functools.py#L653)，可以发现`cache`是`lru_cache`的一个特例：
```python
def cache(user_function, /):
    'Simple lightweight unbounded cache.  Sometimes called "memoize".'
    return lru_cache(maxsize=None)(user_function)
```
而`lru_cache` 的函数签名如下：
```python
def lru_cache(maxsize=128, typed=False):
```
因此可以看出`cache=lru_cache(maxsize=None, typed=False)` 。因此`cache`函数有两个重要的特点：
1. 缓存空间无限大，也就是说不存在缓存的字典的key值超过上限，需要替换掉那些最不常用的key的情况，可以保证所有函数都能命中，但代价是会占用更多的内存。
2. typed=False，表明不同类型的具有相同类型的数值会被当作一个值来缓存。

### 5. 不适合的应用场景
1. **返回可变对象**（如列表、字典）时，缓存的是对象引用，可能导致意外修改
2. **函数有副作用**（如写入文件、修改全局变量）
3. **参数不可哈希**（如传递字典、列表等可变类型）
4. **参数组合可能性无限**（导致缓存无限膨胀）
