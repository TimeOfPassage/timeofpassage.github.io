# ApiSix网关

## 1. Etcd



## 2. ApiSix



## 3. ApiSix-Dashboard

```bash
 docker run --name apisixdashboard \
 		--net=apisix-quickstart-net \
 		-p 9000:9000 \
 		-v /Users/charles.h/apisix/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml \
 		--restart=always \
 		-d apache/apisix-dashboard
```

