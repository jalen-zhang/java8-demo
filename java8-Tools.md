#### Collection.removeIf(Predicate<? super E> filter)

```java
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(str -> str.length()>3); // 删除长度大于3的元素
```

#### Collections.replaceAll(UnaryOperator<E> operator)

```
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(str -> {
    if(str.length()>3)
        return str.toUpperCase();
    return str;
});
```

#### List.sort(Comparator<? super E> c)

```java
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.sort((str1, str2) -> str1.length()-str2.length());
```

#### Map.getOrDefault(Object key, V defaultValue)

```java
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
System.out.println(map.getOrDefault(4, "NoValue"));
```

#### Map.putIfAbsent(K key, V value)

#### Map.remove() | replace() | replaceAll() | merge() | compute() | computeIfAbsent() | computeIfPresent()

