## Trafik-Http教程解析，应用发布的完全适用入门教程

traefik：HTTP层路由，官网：http://traefik.cn/，文档：https://docs.traefik.io/user-guide/kubernetes/

功能和nginx ingress类似。

　　相对于nginx ingress，traefix能够实时跟Kubernetes API 交互，感知后端 Service、Pod 变化，自动更新配置并热重载。Traefik 更快速更方便，同时支持更多的特性，使反向代理、负载均衡更直接更高效。

    ---
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: prom-test1-web  #根据个人喜好随意定义
      namespace: kube-system  #选择服务定义的命名空间
      annotations:
        kubernetes.io/ingress.class: traefik
    spec:
      rules:
      - host: prom.test1.k8s   #声明定义自己申请的域名或者测试使用的域名
	    http:
	      paths:
	      - path: /   #根据自己发布的应用定义空间路径
	    backend:
	      serviceName: prometheus  #查看使用定义的服务名
	      servicePort: 8080        #根据已知服务暴露的端口进行填写


我提供了一个用于服务发布的测试yaml文件，treafik-test.yaml,可以根据服务发布应用结合上文字段说明进行分析和改写测试。

    $kubectl create -f traefix-test.yaml 

之后我们可以进行端口查看和服务应用访问。

查看端口方式：

    $kubectl get ingress -ndefault

之后访问http://域名  或者  https://域名。

这样我们就完成了treafik的路由访问！
