# 구성
| 호스트   | 포트  | 설명                        | url                        | 계정                 |
| -------- | ----- | --------------------------- | -------------------------- | -------------------- |
| CSDLAN01 | 18000 | 로컬 anaconda 환경 juypter  | http://10.120.4.106:18000/ | lotte / Clf4949#     |
| CSDLAN01 | 18010 | docker  컨테이너 위 jupyter | http://10.120.4.106:18010/ | kyeonghyeon.so / 비밀번호1!      |
| CSDLAN02 | 18000 | 로컬 anaconda 환경 juypter  | http://10.120.4.107:18000/ | lotte / Clf4949#     |
| CSDLAN02 | 18010 | docker  컨테이너 위 jupyter | http://10.120.4.107:18010/ | test_user / Clf4949# | 

# Trouble
## HTTP: 403 Forbidden
### 상황
106 서버 죽은 뒤 로컬 jupyterhub을 살리려고하니 에러
대충 아래와 같은 에러메시지
```bash
root@johnjesus-HCL-Desktop:~# jupyterhub
[W 2018-05-15 14:38:10.001 JupyterHub app:366] JupyterHub.proxy_cmd is deprecated in JupyterHub 0.8, use ConfigurableHTTPProxy.command
[I 2018-05-15 14:38:10.002 JupyterHub app:834] Loading cookie_secret from /home/johnjesus/jupyterhub_cookie_secret
[I 2018-05-15 14:38:10.077 JupyterHub app:1528] Hub API listening on http://127.0.0.1:8081/hub/
[W 2018-05-15 14:38:10.079 JupyterHub proxy:415] 
    Generating CONFIGPROXY_AUTH_TOKEN. Restarting the Hub will require restarting the proxy.
    Set CONFIGPROXY_AUTH_TOKEN env or JupyterHub.proxy_auth_token config to avoid this message.

[I 2018-05-15 14:38:10.079 JupyterHub proxy:458] Starting proxy @ https://*:8000/
[E 2018-05-15 14:38:10.094 JupyterHub app:1623]
    Traceback (most recent call last):
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/app.py", line 1621, in launch_instance_async
        yield self.start()
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/app.py", line 1569, in start
        yield self.proxy.check_routes(self.users, self._service_map)
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/proxy.py", line 294, in check_routes
        routes = yield self.get_all_routes()
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/proxy.py", line 589, in get_all_routes
        resp = yield self.api_request('', client=client)
    tornado.httpclient.HTTPError: HTTP 403: Forbidden

root@johnjesus-HCL-Desktop:~# 14:38:10.592 - info: [ConfigProxy] Proxying https://*:8000 to (no default)
14:38:10.595 - info: [ConfigProxy] Proxy API at http://127.0.0.1:8001/api/routes
14:38:10.597 - error: [ConfigProxy] Uncaught Exception Error: listen EADDRINUSE :::8000
    at Object._errnoException (util.js:1022:11)
    at _exceptionWithHostPort (util.js:1044:20)
    at Server.setupListenHandle [as _listen2] (net.js:1367:14)
    at listenInCluster (net.js:1408:12)
    at Server.listen (net.js:1492:7)
    at Object.<anonymous> (/usr/local/lib/node_modules/configurable-http-proxy/bin/configurable-http-proxy:253:19)
    at Module._compile (module.js:652:30)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
    at tryModuleLoad (module.js:505:12)
14:38:10.598 - error: [ConfigProxy] Uncaught Exception Error: listen EADDRINUSE 127.0.0.1:8001
    at Object._errnoException (util.js:1022:11)
    at _exceptionWithHostPort (util.js:1044:20)
    at Server.setupListenHandle [as _listen2] (net.js:1367:14)
    at listenInCluster (net.js:1408:12)
    at doListen (net.js:1517:7)
    at _combinedTickCallback (internal/process/next_tick.js:141:11)
    at Immediate._tickCallback [as _onImmediate] (internal/process/next_tick.js:180:9)
    at runCallback (timers.js:794:20)
    at tryOnImmediate (timers.js:752:5)
    at processImmediate [as _immediateCallback] (timers.js:729:5)
```

### 원인
[출처](https://github.com/jupyterhub/jupyterhub/issues/1377)
![[Pasted image 20221006095403.png]]
### 해결
[출처](https://stackoverflow.com/questions/50346651/jupyterhub-service-unavailable-error-and-http-403-forbidden)
![[Pasted image 20221006095441.png]]
