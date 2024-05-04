# #用Guava实现多线程处理任务

​`定义`​

```java
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newCachedThreadPool());
public Map<Integer, User> multiThreadExecute(List<Integer> ids, Function<List<Integer>, List<User>> function) {
    List<ListenableFuture<List<User>>> results = new ArrayList<>();
    List<List<Integer>> parts = Lists.partition(ids, 5);
    for (List<Integer> pIds : parts) {
        results.add(service.submit(() -> function.apply(pIds)));
    }
    ListenableFuture<List<List<User>>> lf = Futures.allAsList(results);
    try {
        List<List<User>> tmpList = lf.get();
        Map<Integer, User> userMap = new HashMap<>();
        for (List<User> users : tmpList) {
            for (User user : users) {
                userMap.put(user.getId(), user);
            }
        }
        return userMap;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

​`调用`​

```java
public void testMultiThreadExecute() throws Exception {
    List<Integer> ids = Arrays.asList(19, 20);
    Map<Integer, User> userMap = multiThreadExecute(ids, (pIds) -> userRepository.findAllById(pIds));
    System.out.println(userMap.values());
}
```
