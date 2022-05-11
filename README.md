# Example Repo

In this repo I what to play around with ngnix and how we can set it up with the Camunda 8 Platform Helm charts. It is far from complete and more like an POC.

## Add ngnix helm repo


```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
```

## Install ngnix controller


```
helm install zell-ingres-helm-test nginx-stable/nginx-ingress
```

## Install camunda 8 charts


```
helm install camunda-platform-test camunda/camunda-platform -f values.txt
```

## Verify Ingress setup

Ingress should be runnig, you can verify under kubectl get svc, there is an ingress service with an external IP. 

```
kubectl get ingress
```

Should show you all deployed ingresses.

```
$ k get ingress
NAME                             CLASS   HOSTS              ADDRESS          PORTS   AGE
camunda-platform-tes             nginx   keycloak.demo.io   35.205.224.118   80      18m
camunda-platform-test-operate    nginx   operate.demo.io    35.205.224.118   80      18m
camunda-platform-test-optimize   nginx   optimize.demo.io   35.205.224.118   80      18m
camunda-platform-test-tasklist   nginx   tasklist.demo.io   35.205.224.118   80      18m
```

Accessing with the browser is not possible right now with default ngnix configuration. I guess because we have to enable some more things and expose the domain.

To verify whether the setup in general with ingress works you can use cURL, were we can set the Host header.

Example:

```
$ curl -Lv 35.205.224.118:80/api/login -H 'Host: tasklist.demo.io'
```

You will see something like:

```
$ curl -Lv 35.205.224.118:80/api/login -H 'Host: tasklist.demo.io'
*   Trying 35.205.224.118:80...
* Connected to 35.205.224.118 (35.205.224.118) port 80 (#0)
> GET /api/login HTTP/1.1
> Host: tasklist.demo.io
> User-Agent: curl/7.83.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 302 
< Server: nginx/1.21.6
< Date: Wed, 11 May 2022 12:08:46 GMT
< Content-Length: 0
< Connection: keep-alive
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< Location: http://keycloak.demo.io/auth/realms/camunda-platform/protocol/openid-connect/auth?client_id=tasklist&redirect_uri=http%3A%2F%2Ftasklist.demo.io%2Fidentity-callback&response_type=code&scope=openid+email&state=
< Content-Language: en-US
< 
* Connection #0 to host 35.205.224.118 left intact
* Issue another request to this URL: 'http://keycloak.demo.io/auth/realms/camunda-platform/protocol/openid-connect/auth?client_id=tasklist&redirect_uri=http%3A%2F%2Ftasklist.demo.io%2Fidentity-callback&response_type=code&scope=openid+email&state='
* Could not resolve host: keycloak.demo.io
* Closing connection 1
curl: (6) Could not resolve host: keycloak.demo.io
```

If we remove the header you will immediately see an 404, the same what you see in a Browser.

```
$ curl -Lv 35.205.224.118:80/api/login
*   Trying 35.205.224.118:80...
* Connected to 35.205.224.118 (35.205.224.118) port 80 (#0)
> GET /api/login HTTP/1.1
> Host: 35.205.224.118
> User-Agent: curl/7.83.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Server: nginx/1.21.6
< Date: Wed, 11 May 2022 12:09:36 GMT
< Content-Type: text/html
< Content-Length: 153
< Connection: keep-alive
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.21.6</center>
</body>
</html>
* Connection #0 to host 35.205.224.118 left intact
```

TODO: I haven't found a good way to expose it and to make it accessible via Browser.
