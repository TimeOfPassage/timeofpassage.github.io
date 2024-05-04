# Python操作Redis

# 安装包依赖

> pip install redis

## 使用


```python
import redis

# 创建 Redis 连接
r = redis.Redis(host='localhost', port=6379, db=0, password="password")


keys = r.keys("prefix:*")

for key_to_delete in keys:
    # 删除指定的键
    result = r.delete(key_to_delete)
    if result == 1:
        print(f"Key '{key_to_delete}' deleted successfully!")
    elif result == 0:
        print(f"Key '{key_to_delete}' does not exist.")
    else:
        print("Error deleting key.")

```