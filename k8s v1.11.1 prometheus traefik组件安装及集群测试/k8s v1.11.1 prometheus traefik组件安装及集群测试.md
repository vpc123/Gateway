## k8s v1.11.1-1.13.1 prometheus traefik组件安装及集群测试


### 1、traefik

　　traefik：HTTP层路由，官网：http://traefik.cn/，文档：https://docs.traefik.io/user-guide/kubernetes/

　　功能和nginx ingress类似。

　　相对于nginx ingress，traefix能够实时跟Kubernetes API 交互，感知后端 Service、Pod 变化，自动更新配置并热重载。Traefik 更快速更方便，同时支持更多的特性，使反向代理、负载均衡更直接更高效。

　　k8s集群部署Traefik，结合上一篇文章。

　　创建k8s-master-lb的证书：

	[root@k8s-master01 ~]#mkdir -p /cert/traefik
	[root@k8s-master01 ~]#cd /cert/traefik
    [root@k8s-master01 ~]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=k8s-master-lb"
    Generating a 2048 bit RSA private key
    ................................................................................................................+++
    .........................................................................................................................................................+++
    writing new private key to 'tls.key'


	把证书写入到k8s的secret

    [root@k8s-master01 ~]# kubectl -n kube-system create secret generic traefik-cert --from-file=tls.key --from-file=tls.crt
    secret/traefik-cert created

	安装traefik

    [root@k8s-master01 kubeadm-ha]# kubectl apply -f traefik/
    serviceaccount/traefik-ingress-controller created
    clusterrole.rbac.authorization.k8s.io/traefik-ingress-controller created
    clusterrolebinding.rbac.authorization.k8s.io/traefik-ingress-controller created
    configmap/traefik-conf created
    daemonset.extensions/traefik-ingress-controller created
    service/traefik-web-ui created
    ingress.extensions/traefik-jenkins created


查看pods，因为创建的类型是DaemonSet所有每个节点都会创建一个Traefix的pod

    [root@k8s-master01 kubeadm-ha]# kubectl  get pods -nkube-system

	traefik-ingress-controller-4qkjb       1/1     Running   0          69s     10.244.1.10      k8s-node01   <none>           <none>
	traefik-ingress-controller-v5sc4       1/1     Running   0          69s     10.244.0.9       k8s-master   <none>           <none>


打开Traefix的Web UI：http://k8s-master-lb:30011/


创建测试web应用

    [root@k8s-master01 ~]# kubectl create -f traefix-test.yaml 
    service/nginx-svc created
    deployment.apps/ngx-pod created
    ingress.extensions/ngx-ing created

HTTPS证书配置

利用上述创建的nginx，再次创建https的ingress

    [root@k8s-master01 nginx-cert]# cat ../traefix-https.yaml 
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: nginx-https-test
      namespace: default
      annotations:
    kubernetes.io/ingress.class: traefik
    spec:
      rules:
      - host: traefix-test.com
    http:
      paths:
      - backend:
      serviceName: nginx-svc
      servicePort: 80
      tls:
       - secretName: nginx-test-tls


　创建证书，线上为公司购买的证书

    [root@k8s-master01 nginx-cert]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=traefix-test.com"
    Generating a 2048 bit RSA private key
    .................................+++
    .........................................................+++
    writing new private key to 'tls.key'
    -----



导入证书

    $kubectl -n default create secret tls nginx-test-tls --key=tls.key --cert=tls.crt


创建ingress

    [root@k8s-master01 ~]# kubectl create -f traefix-https.yaml 
    ingress.extensions/nginx-https-test created


查看已经存在的ingress，并且根据ingress修改访问主机的hosts文件，使其可以正常访问网页服务。

    $kubectl get ingress -nkube-system
	$vi /etc/hosts

其他方法查看官方文档：https://docs.traefik.io/user-guide/kubernetes/