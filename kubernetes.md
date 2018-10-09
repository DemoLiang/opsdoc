# k8s 安装过程

阐述的是二进制安装方式

1. 先消除防火墙影响，取消selinux,让容器可以去读取主机文件系统
```
    systemctl disable firewalld
    systemctl stop firewalld

    setenforce 0
```

2. master节点安装
 
    - 下载相应的kubernetes server binary,etcd 二进制文件
    - cp etcd kube-apiserver,kube-controller-manager,kube-scheduler etcdctl  到/usr/bin
    - 写入etcd 启动配置信息
    ```
        #cat /usr/lib/systemd/system/etcd.service 

        [Unit]
        Description=Etcd Server
        After=network.target

        [Service]
        Type=simple
        WorkingDirectory=/var/lib/etcd/
        EnvironmentFile=-/etc/etcd/etcd.conf
        ExecStart=/usr/bin/etcd

        [Install]
        WantedBy=multi-user.target

    ```

    - 启动etcd 
    ```
    systemctl daemon-reload
    systemctl enable etcd.service
    systemctl start etcd.service

    检查启动情况

    systemctl status etcd.service

    etcdctl cluster-health
    ```

    - 写入kube-apiserver 配置文件并启动
    ```
    #cat /usr/lib/systemd/system/kube-apiserver.service 
    
    [Unit]
    Description=Kubernetes API Server
    Documentation=https://github.com/GoogleCloudPlatform/Kubernetes
    After=etcd.service
    Wants=etcd.service

    [Service]
    EnvironmentFile=/etc/kubernetes/apiserver
    ExecStart=/usr/bin/kube-apiserver $KUBE_API_ARGS
    Restart=on-failure
    Type=notify
    LimitNOFILE=65536

    [Install]
    WantedBy=muti-user.target
    ```

    ```
    # cat /etc/kubernetes/apiserver 
    
    KUBE_API_ARGS="--storage-backend=etcd3 --etcd-servers=http://127.0.0.1:2379 --insecure-bind-address=0.0.0.0 --insecure-port=8080 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
    ```

    - 写入kube-controller-manager 配置文件并启动
    ```
    # cat /usr/lib/systemd/system/kube-controller-manager.service 
    
    [Unit]
    Description=Kubernetes Controller Manager
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes
    After=kube-apiserver.service
    Requires=kube-apiserver.service

    [Service]
    EnvironmentFile=/etc/kubernetes/controller-manager
    ExecStart=/usr/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
    Restart=on-failure
    LimitNOFILE=65536

    [Install]
    WantedBy=multi-user.target
    ```
    ```
    # cat /etc/kubernetes/controller-manager 
    
    KUBE_CONTROLLER_MANAGER_ARGS="--master=http://172.16.250.170:8080 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
    ```
    - 写入kube-scheduler 配置文件并启动
    ```
        # cat /usr/lib/systemd/system/kube-scheduler.service 

        [Unit]
        Description=Kubernetes Controller Manager
        Documentation=https://github.com/GoogleCloudPlatform/kubernetes
        After=kube-apiserver.service
        Requires=kube-apiserver.service

        [Service]
        EnvironmentFile=/etc/kubernetes/scheduler
        ExecStart=/usr/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
        Restart=on-failure
        LimitNOFILE=65536

        [Install]
        WantedBy=muti-user.target
    ```
    ```
        # cat /etc/kubernetes/scheduler 
        
        KUBE_SCHEDULER_ARGS="--master=http://172.16.250.170:8080 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
    ```

3. node 节点安装，目前是直接在master机器上直接安装

    - 安装docker服务
    ```
    yum install -y dokcer
    ```
    - 安装配置kubelet 服务
        注意目前已经不支持 --api-servers 参数，请使用--kubeconfig
        注意参数要加上swap=false 
        注意更改docker driver参数 --cgroup-driver=systemd
    
    ```
    # cat /usr/lib/systemd/system/kubelet.service 
    
    [Unit]
    Description=Kubernetes Kubelet Server
    Documentation=https://github.com/GoogleCloudOlatform/kubernetes
    After=docker.service
    Requires=docker.service

    [Service]
    WorkingDirectory=/var/lib/kubelet
    EnvironmentFile=/etc/kubernetes/kubelet
    ExecStart=/usr/bin/kubelet $KUBELET_ARGS
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target
    ```
    ```
    # cat /etc/kubernetes/kubelet 
    
    KUBELET_ARGS="--kubeconfig=/etc/kubernetes/master-kubeconfig.yaml --hostname-override=172.16.250.170 --fail-swap-on=false --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
    ```
    ```
    # cat /etc/kubernetes/master-kubeconfig.yaml 

    apiVersion: v1
    kind: Config
    clusters:
    - name: local
    cluster:
        server: http://127.0.0.1:8080
    users:
    - name: kubelet
    contexts:
    - context:
        cluster: local
        user: kubelet
    name: kubelet-context
    current-context: kubelet-context
    ```