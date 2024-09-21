# map 和 unordered_map 使用实践

## 0. 简介
[map](https://en.cppreference.com/w/cpp/container/map) 和[unordered_map](https://en.cppreference.com/w/cpp/container/unordered_map) 的对于用户的使用方式上是相同的，都是一个key, value的查找表。但是在使用场景和效率上有所不同。
## 1. map 和 unordered_map 不同点
### 1.1 使用场景不同
map 是内部有序的，按照key的值进行从小到大排序（可以自定义排序方式）。所以可以通过获取首个元素获取key值最小的对应的value值，或者通过find找到对应key值相邻大小的<key,value>。常用方法可见示例。

unordered_map内部是无序的，所以对于用户来说是一个纯粹的查找表：根据key值找到对应的vlue.


### 1.2 性能与资源消耗不同
通常情况下，map 的平局算法复杂度 O(logN), un_ordered_map 的算法复杂都O(1)。但是un_ordered_map 因为是散列存储，所以消耗的存储资源要比 map 高（线性比例）。
另外，如果是频繁使用插入和删除等操作的时候，性能消耗将会增加。

### 1.3 实现原理不同
map: 内部实现了一个红黑树，该结构具有自动排序的功能，因此map内部的所有元素都是有序的，红黑树的每一个节点都代表着map的一个元素，因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行这样的操作，故红黑树的效率决定了map的效率。
unordered_map: 内部实现了一个哈希表，因此其元素的排列顺序是杂乱的，无序的


### 1.4 使用 map or unordered_map ?
根据stack overflow 上的讨论 [Is there any advantage of using map over unordered_map in case of trivial keys?](https://stackoverflow.com/questions/2196995/is-there-any-advantage-of-using-map-over-unordered-map-in-case-of-trivial-keys) 总结几点使用map的场景如下：
1. 如果需要key的排序功能场景，使用map
2. 频繁的插入，删除节点的场景，使用map

否则在使用 unordered_map 效率会更高。

## 2 使用典型示例
### 2.1 map
模板结构：
```c++
template<
    class Key,
    class T,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<std::pair<const Key, T>>
> class map;
```
#### 2.1.1 默认有序存储
```c++
#include <iostream>
#include <map>
#include <memory>
void separator() { std::cout << std::endl << "------" << std::endl; }

int main() {

  // 构建map <1, "first">, <2, "second">, <3, "third">, <5, "fourth">
  std::map<int, std::string> m_map;
  m_map.emplace(2, "second");
  m_map.emplace(1, "first");
  m_map.emplace(5, "fourth");
  m_map.emplace(3, "third");

  // 自动按key从小到大排序（而不是数据插入顺序）： first second third fourth
  for (auto it = m_map.begin(); it != m_map.end(); it++) {
    std::cout << it->second << " ";
  }
  separator();

  // 获取最小的key的value值，预期打印: <1, "first">
  std::cout << m_map.begin()->first << "," << m_map.begin()->second;
  separator();

  // 获取最大的key的value值，预期打印: <5, "fourth">
  auto max_iter = m_map.end();
  max_iter--;
  std::cout << max_iter->first << "," << max_iter->second;
  separator();

  // 获取key不大于4的最大值，预期打印: <3, "third">
  auto iter = m_map.upper_bound(4);
  if (iter != m_map.begin()) {
    iter--;
    std::cout << iter->first << "," << iter->second;
  } else {
    std::cout << "not found.";
  }
  separator();

  return 0;
}

```
#### 2.1.2 自定义比较器
```c++
// 构建自己的比较器，如果类型确定，可以不使用模板，此处定义从大到小的排列
template<class T>
struct MyGreaterCompare {
  constexpr bool operator()(const T &lhs, const T &rhs) const {
    // 如果是类可以自己定义
    return lhs > rhs;
  }
};

// 构建map <5, "fourth">, <3, "third">, <2, "second">, <1, "first">
  std::map<int, std::string, MyGreaterCompare<int>> m_map;
  m_map.emplace(2, "second");
  m_map.emplace(1, "first");
  m_map.emplace(5, "fourth");
  m_map.emplace(3, "third");

  // 自动按key从小到大排序（而不是数据插入顺序）： fourth third second first
  for (auto it = m_map.begin(); it != m_map.end(); it++) {
    std::cout << it->second << " ";
  }
```
### 2.2 unordered_map
模板结构
```c++
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<std::pair<const Key, T>>
> class unordered_map;
```

#### 2.2.1 无序存储
```c++
#include <iostream>
#include <memory>
#include <unordered_map>

void separator() { std::cout << std::endl << "------" << std::endl; }

int main() {
  // 构建map <2, "second">, <1, "first">, <5, "fourth">, <3, "third">
  std::unordered_map<int, std::string> m_map;
  m_map.emplace(2, "second");
  m_map.emplace(1, "first");
  m_map.emplace(5, "fourth");
  m_map.emplace(3, "third");

  // 无序，数据插入顺序： second first fourth third
  for (auto it = m_map.begin(); it != m_map.end(); it++) {
    std::cout << it->second << " ";
  }
  separator();

  // 获取bucket 个数
  std::cout << m_map.bucket_count();
  separator();
  return 0;
}
```
#### 2.2.2 自定义hash函数
引用[std::hash](https://en.cppreference.com/w/cpp/utility/hash)
```c++
#include <cstddef>
#include <functional>
#include <iomanip>
#include <iostream>
#include <string>
#include <unordered_set>
 
struct S {
  std::string first_name;
  std::string last_name;
  // 此处可以自定义比较器
  bool operator==(const S &rhs) const noexcept{
    return first_name == rhs.first_name && last_name == rhs.last_name;
  }
  S(std::string first, std::string second):first_name(first),last_name(second){}
};

struct MyHash {
  std::size_t operator()(const S &s) const noexcept {
    std::size_t h1 = std::hash<std::string>{}(s.first_name);
    std::size_t h2 = std::hash<std::string>{}(s.last_name);
    return h1 ^ (h2 << 1); // or use boost::hash_combine
  }
};
 
int main()
{
  //定义时候传入hash函数对象
  std::unordered_map<S, std::string, MyHash> m_map;
  m_map.emplace(std::piecewise_construct, std::forward_as_tuple("b", "bb"),
                std::forward_as_tuple("second"));
  m_map.emplace(std::piecewise_construct, std::forward_as_tuple("a", "aa"),
                std::forward_as_tuple("first"));
  // 无序，数据插入顺序： second first fourth third
  for (auto it = m_map.begin(); it != m_map.end(); it++) {
    std::cout << it->second << " ";
  }
  separator();

  // 获取bucket 个数
  std::cout << m_map.bucket_count();
}
```