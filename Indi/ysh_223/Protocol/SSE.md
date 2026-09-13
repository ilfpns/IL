|SSE
[[Protocol]]
- 클라이언트는 데이터를 받을 수만 있다
- Socket-Sent-Event로 서버는 데이터를 받지 않는다
- 3초 간격으로 재접속을 시도한다
- 최대 6개의 HTTP 브라우저를 접속시킬 수 있다
- 배터리 소모량이  작다 
  -  Websocket에 비해 보내는 양이 적음

=> SSE를 대체하기 위해 Short Polling이 나오게 됨
	- 서버의 호출 주기를 짧게 해, SSE를 구현함
	  => 필요 이상으로 요청을 해 HTTP overhead가 일어난다

-  이를 대체한 Long Polling이 출시됨
	 => 클라이언트의 요청을 받고 바로 응답하지 않고,  값이 바뀌면 응답