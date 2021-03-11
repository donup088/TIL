### 특정 port 찾아서 중단시키기
```
netstat -nap | grep 8080
```
```
netstat -anp | grep LISTEN | grep :포트번호
kill -9 PID
```