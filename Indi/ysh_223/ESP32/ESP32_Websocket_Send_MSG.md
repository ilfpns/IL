[[ESP32]]
### Json 전송

```c
void websocket_send_msg(void)
{
    if (!esp_websocket_client_is_connected(client))
    {
        ESP_LOGE(TAG, "WS not connected");
        return;
    }

    char *jsonUserId = "{\\"id\\":1, \\"value\\":1}";

    int sendLen = esp_websocket_client_send_text(client, jsonUserId, strlen(jsonUserId), portMAX_DELAY);

        if (sendLen <= 0)
            ESP_LOGE(TAG, "WS msg send failed");
    else ESP_LOGI(TAG, "sent %d bytes msg", sendLen);
}
```

1. 핸들러 연결 확인
    
2. 보낼 데이터를 변수에 담기
    
3. `esp_websocket_client_send_text` 로 (핸들러, 보낼 데이터, 총 길이 데이터, 기다릴 시간)
    
    ⇒ strlen으로 길이를 딱 맞추어 메모리 누수 방지
    
    ⇒ `portMAX_DELAY` 반환값이 돌아올 때까지 무한정 기다리기
    
4. 반환값으로 보낸 bytes 길이