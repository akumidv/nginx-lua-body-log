## About
Nginx proxy body logger

## Build

`docker build -t nginx_lua .`

## Run

Create server.conf from example
```bash

server {
  listen 80 default_server;

  lua_need_request_body on;

  set $resp_body "";
  set $req_body "";
  set $req_headers "";

  client_body_buffer_size 16k;
  client_max_body_size 16k;

  rewrite_by_lua_block {
            local req_headers = "Headers: ";
            ngx.var.req_body = ngx.req.get_body_data();
            local h, err = ngx.req.get_headers()
            for k, v in pairs(h) do
                req_headers = req_headers .. k .. ": " .. v .. "\n";
            end

            ngx.var.req_headers = req_headers;
  }

  body_filter_by_lua '
        local resp_body = string.sub(ngx.arg[1], 1, 1000)
        ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
        if ngx.arg[2] then
          ngx.var.resp_body = ngx.ctx.buffered
        end
  ';


  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://iondv:8888;
  }
  access_log  /var/log/nginx/req-resp.log bodylog;


```

Replace http://iondv:8888 to your url.

Start `docker run -v /workspace/nginx-lua/iondv.conf.example:/etc/nginx/conf.d/default.conf:ro -v body_log:/var/log/nginx -p "80:80" --link iondv --rm --name nginx_lua nginx_lua`, where `--link iondv` docker name - skip if you hav url or replace to your docker name.

On ubuntu `cat /var/lib/docker/volumes/body_log/_data/req-resp.log` fog reques/response log.

# Source 
https://gist.github.com/morhekil/1ff0e902ed4de2adcb7a
