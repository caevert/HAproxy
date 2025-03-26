# HAproxy

[Файлы конфигурации](files/)


Дашборд Grafana
![Dashbord-1](./files/Graf1.png)

![Dashbord-2](./files/Graf2.png)
Метрики Grafana
![Dashbord-3](./files/Graf_metrics.png)

haproxy.cfg

```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-PO>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http
        log-format {\"type\":\ \"haproxy\",\"client_ip\":\ \"%ci\",\"client_port\":\ \"%cp\",\"request_date\":\ \"[%t]\",\"frontend_name\":\ \"%f\",\"backend_name\":\ \"%b\",\"server_name\":\ \"%s\",\>

cache rebrain_cache
       total-max-size 128
       max-object-size 10000
       max-age 30

frontend rebrain_front
       bind *:443 ssl crt /etc/haproxy/rebrain.pem
       mode http
       acl url_api path_beg /api
       acl url_lk path_beg /lk
       option forwardfor
       http-request set-header X-Forwarded-For %[src]
       http-request cache-use rebrain_cache
       http-response cache-store rebrain_cache
       use_backend rebrain_api if url_api
       use_backend rebrain_lk if url_lk
       default_backend rebrain_back

frontend front_sql
       bind *:3307
       mode tcp
       option tcplog
       default_backend rebrain_sql

backend rebrain_api
       mode http
       balance roundrobin
       option prefer-last-server
       cookie REBRAIN insert indirect nocache
       server rebrain_01_80 127.0.0.1:80 cookie rebrain_01_80 check
       server rebrain_02_80 127.0.0.1:80 cookie rebrain_02_80 check

backend rebrain_lk
       mode http
       balance leastconn
       acl is_cached path_end -i .js .php .css
       http-request cache-use rebrain_cache if is_cached
       http-response cache-store rebrain_cache if is_cached
       server rebrain_01_81 127.0.0.1:81 check inter 4s
       server rebrain_02_81 127.0.0.1:81 check inter 4s maxconn 80

backend rebrain_back
       mode http
       balance source
       cookie PHPSESSID prefix
       server rebrain_01_82 127.0.0.1:82 cookie s1 check port 82 inter 8s maxconn 1100
       server rebrain_02_82 127.0.0.1:82 cookie s2 check port 82 inter 8s maxconn 1100
       server rebrain_03_82 127.0.0.1:82 cookie s3 check port 82 inter 8s maxconn 1100

backend rebrain_sql
       mode tcp
       balance roundrobin
       option mysql-check user haproxy
       server rebrain_db_1 127.0.0.1:3306 check port 3306 inter 2s fall 2 rise 1 maxconn 100
       server rebrain_db_2 127.0.0.1:3306 check port 3306 inter 2s fall 2 rise 1 maxconn 100

listen stat
       bind *:777
       mode http
       stats enable
       stats hide-version
       stats realm Haproxy Statistics
       stats uri /
       stats auth admin:admin
       stats refresh 30s



```shell
cd kubespray
cp inventory/sample inventory/netology
```

3. Выясняем айпи машин кластера на которые будет производится установка:

![Kube](./assets/K-1.png)
