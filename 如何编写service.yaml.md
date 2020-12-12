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
        port: 3306
        targetPort: 3306
      - name: tcp-33060
        protocol: TCP
        port: 33060
        targetPort: 33060

  ```
