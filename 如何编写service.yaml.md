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
      - name: tcp-3306 # 关联容器暴露端口名，见deploy.yaml
        protocol: TCP
        port: 3306 # pod暴露
        targetPort: 3306 # service暴露端口
        nodePort: 20001 # k8s暴露端口
      - name: tcp-33060
        protocol: TCP
        port: 33060
        targetPort: 33060

  ```
