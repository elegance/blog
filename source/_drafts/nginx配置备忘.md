---
title: nginx配置备忘
tags:
---

### 根据 Referer 动态 proxy
```
        location /tzb-api {
            if ($http_referer ~ "/simu-app/") { # 私募 访问 2.0 api
                #proxy_pass http://10.26.255.220:8080/tzb-api;  不可写固定的地址，nginx 会提示检测不通过
                proxy_pass http://10.26.255.220:8080$request_uri;
            }
            if ($http_referer !~ "/simu-app/") {
                proxy_pass http://localhost:8089$request_uri;  # 保留原访问 1.0 api
            }
            proxy_set_header  X-Real-IP  $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header  Host  $http_host;
        }
```

