```c
// 트리거를 받는 쪽
ulTaskNotifyTake(dht, portMAX_DELAY);
// 트리거를 실행시키는 TASK
xTasknotifyGive(dht);
```

⇒ 트리거 역할을 하며, 하나의 TASK가 다른 TASK를 관리할 수 있다

[[Tag/RTOS]]