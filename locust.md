# Locust

<https://docs.locust.io/en/stable/index.html>

## install

```bash
pip install locust
```

## example

<https://docs.locust.io/en/stable/writing-a-locustfile.html>

```python
"""locustfile.py"""

import time
from locust import HttpUser, task, between

class QuickstartUser(HttpUser):  # 模擬一個 user，每次 spawn init a new instance
    wait_time = between(1, 5)  # 每次跑完一個 task 會等，如果沒有設定就會馬上執行下一run

    # 每一 run 有 @task 的會隨機被選擇執行，可以自己寫 helper function
  
    @task  
    def hello_world(self):
        self.client.get("/hello")
        self.client.get("/world")

    @task(3)  # 3: sample weight
    def view_items(self):
        for item_id in range(10):
            self.client.get(f"/item?id={item_id}", name="/item")  
            # 用 name 把 request grouping 起來有利於 locust 統計
            time.sleep(1)

    def on_start(self):  # user 被創出來會先跑
        self.client.post("/login", json={"username":"foo", "password":"bar"})
```

## web interface

<http://localhost:8089>
