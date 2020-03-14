## 创建CA私钥 (KEY)
```
openssl genrsa -out ca.key 2048
```

## 创建CA自签名证书 (CRT)
```
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```

## 创建服务器私钥 (KEY)
```
openssl genrsa -out server.key 2048
```

## 创建服务器证书签名请求文件 (CSR)
```
openssl req -new -key server.key -out server.csr
```

## 使用自建CA签发服务器证书（CRT)
```
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```
