# 如何编写deploy.yaml?
- 可参考kubesphere编写
  ```
  kind: Deployment  # 指定kind为Deployment，一个工作负载
  apiVersion: apps/v1 
  metadata:
    name: mysql-master-v1
    namespace: gulimall
    labels: # labels相当于打标签，用于后面引用，可以加入任意key-value
      app: mysql-master
      version: v1
    annotations:
      deployment.kubernetes.io/revision: '1'
      kubesphere.io/creator: saw
  spec:
    progressDeadlineSeconds: 600 # 单位秒，600秒后确定程序已卡主
    replicas: 1 # 指定容器组副本数量。即pod副本数量
    selector:
      matchLabels: # 匹配上述Deployment
        app: mysql-master
        version: v1
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: mysql-master
          version: v1
        annotations:
          kubesphere.io/containerSecrets: ''
          logging.kubesphere.io/logsidecar-config: '{}'
      spec: 
        volumes: # 定义要使用的数据卷
          - name: volume-quhssz # 数据卷名称
            persistentVolumeClaim: # 使用pvc
              claimName: mysql-master-pvc # # 使用对应名称pvc
          - name: volume-k0vveu
            configMap: # 使用configMap
              name: mysql-master-conf # 使用对应名称configMap
              items:
                - key: custom.cnf # 使用对应key
                  path: custom.cnf # 生成对应path。相当于在挂载目录下生成custom.cnf配置文件
              defaultMode: 420 # 文件权限
        containers: # 定义容器组pod中的容器，containers表明可以有多种container
          - name: mysql-master-container
            image: 'mysql:8.0' # 使用$REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME从jenkins从流水线上下文中动态取值
            command: # 可
              - mysqld
            ports: # 声明容器暴露端口
              - name: tcp-3306
                containerPort: 3306
                protocol: TCP
              - name: tcp-33060
                containerPort: 33060
                protocol: TCP
            env: # 相当于-e or --env
              - name: MYSQL_ROOT_PASSWORD
                valueFrom: # 从密钥中获取
                  secretKeyRef:
                    name: mysql-master-security
                    key: MYSQL_ROOT_PASSWORD
            resources: # 分配在k8s中使用的资源
              limits: # 限制
                cpu: 970m
                memory: 2000Mi
              requests: # 初始分配
                cpu: 150m
                memory: 100Mi
            volumeMounts: # 对应挂载至容器的路径和权限
              - name: host-time
                readOnly: true
                mountPath: /etc/localtime
              - name: volume-quhssz
                mountPath: /var/lib/mysql
              - name: volume-k0vveu
                readOnly: true
                mountPath: /etc/mysql/conf.d
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
            imagePullPolicy: IfNotPresent
        restartPolicy: Always
        terminationGracePeriodSeconds: 30
        dnsPolicy: ClusterFirst
        serviceAccountName: default
        serviceAccount: default
        securityContext: {}
        affinity: {}
        schedulerName: default-scheduler
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxUnavailable: 25%
        maxSurge: 25%
    revisionHistoryLimit: 10
    progressDeadlineSeconds: 600

  ```
