# 可参考kubesphere上的服务配置文件
  ```
  apiVersion: v1
  kind: Service
  metadata:
    namespace: gulimall
    labels:
      version: v1
      app: mysql-master
    annotations:
      kubesphere.io/serviceType: statelessservice
      kubesphere.io/alias-name: MySQL Master
      kubesphere.io/description: MySQL主机
    name: mysql-master
  spec:
    sessionAffinity: None
    selector:
      app: mysql-master
    template:
      metadata:
        labels:
          version: v1
          app: mysql-master
    ports:
      - name: tcp-3306
        protocol: TCP
        # 192.168.56.101:20001 => mysql-master:3306 ==DNS负载均衡==> 目标容器:3306。多副本容器不会冲突的原因是因为ip不同，如:
        # 10.244.1.216:3306, 10.244.1.217:3306, 10.244.2.20:3306, 10.244.2.21:3306
        port: 3306 # 服务暴露端口
        targetPort: 3306 # 对应容器暴露端口
        nodePort: 20001 # 暴露给主机，通过任何k8s节点:20001可访问对应服务3306端口对应容器3306端口
      - name: tcp-33060
        protocol: TCP
        port: 33060
        targetPort: 33060

  ```
