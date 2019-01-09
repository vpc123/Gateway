## Traefik基于https的自认证证书进行应用服务发布

关于此文档的yaml同时保存在同级目录下。

    $cat ../traefix-https.yaml 
 
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: nginx-https-test #ingress的名字
      namespace: default     #命名空间
      annotations:
    	kubernetes.io/ingress.class: traefik
    spec:
      rules:
      - host: traefix-test.com   #访问host的域名地址
	    http:
	      paths:
	      - backend:
	      serviceName: nginx-svc #选择的命名空间下的服务名进行映射
	      servicePort: 80        #容器服务开放的端口
	      tls:
	       - secretName: nginx-test-tls   #选择指定生成的秘钥文件,和证书验证



创建证书，线上为公司购买的证书，穷屌丝的我们就是做个试验想那么多干嘛！

    [root@k8s-master01 nginx-cert]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=traefix-test.com"
    Generating a 2048 bit RSA private key
    .................................+++
    .........................................................+++
    writing new private key to 'tls.key'
    -----

注意生成的traefix-test.com  参数需要和文件中定义一致才可以。

导入证书

    $kubectl -n default create secret tls nginx-test-tls --key=tls.key --cert=tls.crt

创建ingress

    [root@k8s-master01 ~]# kubectl create -f traefix-https.yaml 
    ingress.extensions/nginx-https-test created


至此，需要证书验证的网络访问服务既https访问请求的treafik示例已经完全讲解完毕！