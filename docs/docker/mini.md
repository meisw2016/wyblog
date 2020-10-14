# docker部署minIO
~~~
docker run -p 9090:9000 --name minio \
  --net consul --ip 172.19.0.31\
  -e MINIO_ACCESS_KEY=admin -e MINIO_SECRET_KEY=123123123 \
  -v /mydata/minio/data:/data \
  -v /mydata/minio/config:/root/.minio \
  -d minio/minio server /data;# 如果不创建用户名密码，默认用户名密码： minioadmin:minioadmin
~~~
# 访问
访问地址： http://localhost:9090/minio
<br/>
![logo](http://192.169.60.130:9090/images/clipboard.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=admin%2F20201011%2F%2Fs3%2Faws4_request&X-Amz-Date=20201011T150636Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=41cd5051b1502c272c36492d15ad2af5ebe3b07dd9cd3152aec8bfb4d5c53994)
