# HW-12.1

## Задача 1: Установить Minikube

Миникуб установил на отдельную ВМ в среде ВМваре (сервер linux ubuntu)

    oleg@mck-devops-tools:~$ minikube version
    minikube version: v1.25.2
    commit: 362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
    oleg@mck-devops-tools:~$


    root@mck-devops-tools:/home/oleg# minikube status
    minikube
    type: Control Plane
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured

    root@mck-devops-tools:/home/oleg#


    root@mck-devops-tools:/home/oleg# kubectl get pods --namespace=kube-system
    NAME                                       READY   STATUS    RESTARTS   AGE
    coredns-64897985d-h8gbz                    0/1     Running   0          39s
    etcd-mck-devops-tools                      1/1     Running   0          50s
    kube-apiserver-mck-devops-tools            1/1     Running   0          53s
    kube-controller-manager-mck-devops-tools   1/1     Running   0          50s
    kube-proxy-29n9s                           1/1     Running   0          39s
    kube-scheduler-mck-devops-tools            1/1     Running   0          50s
    storage-provisioner                        1/1     Running   0          49s
    root@mck-devops-tools:/home/oleg#


## Задача 2: Запуск Hello World

 ### 1 Запуск дашбоард

    minikube dashboard
    
    kubectl proxy --address="10.102.8.79" -p 8081 --accept-hosts='^*$'

После чего смог запустить дашбоард по ссылке с другого сервера:

http://10.102.8.79:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default

скриншота dashboard

![dashboard](https://github.com/olegrovenskiy/HW-12.1/blob/main/dashboard.png)

 ### 2. Deployment

    root@mck-devops-tools:/etc/kubernetes/minikube# kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
    deployment.apps/hello-node created


    root@mck-devops-tools:/etc/kubernetes/minikube# kubectl get deployments
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    hello-node   1/1     1            1           82s
    root@mck-devops-tools:/etc/kubernetes/minikube#


    root@mck-devops-tools:/etc/kubernetes/minikube# kubectl get pods
    NAME                          READY   STATUS    RESTARTS   AGE
    hello-node-6b89d599b9-54fs2   1/1     Running   0          112s
    root@mck-devops-tools:/etc/kubernetes/minikube#


    root@mck-devops-tools:/etc/kubernetes/minikube# kubectl get events
    LAST SEEN   TYPE     REASON              OBJECT                             MESSAGE
    2m25s       Normal   Scheduled           pod/hello-node-6b89d599b9-54fs2    Successfully assigned default/hello-node-6b89d599b9-54fs2 to mck-devops-tools
    2m25s       Normal   Pulling             pod/hello-node-6b89d599b9-54fs2    Pulling image "k8s.gcr.io/echoserver:1.4"
    2m20s       Normal   Pulled              pod/hello-node-6b89d599b9-54fs2    Successfully pulled image "k8s.gcr.io/echoserver:1.4" in 4.910100893s
    2m18s       Normal   Created             pod/hello-node-6b89d599b9-54fs2    Created container echoserver
    2m18s       Normal   Started             pod/hello-node-6b89d599b9-54fs2    Started container echoserver
    2m25s       Normal   SuccessfulCreate    replicaset/hello-node-6b89d599b9   Created pod: hello-node-6b89d599b9-54fs2
    2m25s       Normal   ScalingReplicaSet   deployment/hello-node              Scaled up replica set hello-node-6b89d599b9 to 1
    root@mck-devops-tools:/etc/kubernetes/minikube#


    root@mck-devops-tools:/etc/kubernetes/minikube# kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: /root/.minikube/ca.crt
        extensions:
        - extension:
            last-update: Mon, 14 Mar 2022 07:03:03 UTC
            provider: minikube.sigs.k8s.io
            version: v1.25.2
          name: cluster_info
        server: https://10.102.8.79:8443
      name: minikube
    contexts:
    - context:
        cluster: minikube
        extensions:
        - extension:
            last-update: Mon, 14 Mar 2022 07:03:03 UTC
            provider: minikube.sigs.k8s.io
            version: v1.25.2
          name: context_info
        namespace: default
        user: minikube
      name: minikube
    current-context: minikube
    kind: Config
    preferences: {}
    users:
    - name: minikube
      user:
        client-certificate: /root/.minikube/profiles/minikube/client.crt
        client-key: /root/.minikube/profiles/minikube/client.key
    root@mck-devops-tools:/etc/kubernetes/minikube#


Создание сервиса

    kubectl expose deployment hello-node --type=LoadBalancer --port=8080

    root@mck-devops-tools:/etc/kubernetes/minikube# kubectl get services
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.109.113.63   <pending>     8080:31031/TCP   6s
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          34h
    root@mck-devops-tools:/etc/kubernetes/minikube#

Получаю ссылку на сервис

    root@mck-devops-tools:/etc/kubernetes/minikube# minikube service hello-node --url
    http://10.102.8.79:31031
    root@mck-devops-tools:/etc/kubernetes/minikube#



## Задача 3: Установить kubectl

Установил на рабочую станцию с Виндоус из документации

https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/

    cd %USERPROFILE%
    mkdir .kube

В неё разместил конфигурационный файл

    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJeU1ETXhNekEzTURJeE4xb1hEVE15TURNeE1UQTNNREl4TjFvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTS92CkJMMmxlSFdmRHBxU2U2cGlraGNNWnRVZ3VvM2cxM29KOEpMZ3Q4WWRlLzRqeERDR2MxcDdrMmFadUNCbHZqbVYKaGxGbkkrYjZnT2RkRDNWUEtuajZvSmtqK3I2RFN6enluNUlzeXd2TXp2NlhzNEVJRVVjeEI2Yk9vdnJTdU44aQpZVzRoSFZOSU5jLzAzdWdQc2lod0VzeU16LzBMb3cycm9sMC9zT3I5TmovbFpSdDdLSmpmdUVwcmxoSHF2bTlSCm1sb04yeEZiZHJPeHZVZ2xzWEVEL2M1S3F1bjdlWE41enNMbkF0eVJWdUtvYzZ1NzQ0NFg5WkJxMlB0c1c0NW8Ka01LSFBDbXhXNSs4Y3FrbFhmbGxIdW1GL1JWb3ZWMkl5OGZ6L3psQ1FzSkxNQVRiMytDVmZZMkwzUzJPaGNaSQp5M2o3bVozdThWZURVaVJMa3JrQ0F3RUFBYU5oTUY4d0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjBHQTFVZERnUVcKQkJRWHVQWHRaZTVOR3NEOGI0REV6K05VeHpRZU9qQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFzTElKcWpOUwoxUHJDZ1RpSUJSRmpLMmFQa29EWU14UlMrZ1BqQ0FlWVF2bE9xY0FWaGVIem9YZEtUeTVLUXI2bk9laGV3VWt6CkVsbUVmRzladzZsVWxOYXF5QlYxanFLdmlyUFVJYmJnSURMVmtuMUN1eEhtemxLdFJxcVJFdjZETm05UkRpcUUKK25McFhtRUZKZG0zdjVHdjFLNUNrV1JiaFZZQ28wQjh4ZW80aE9oVHh2OU0wZWF5aWtxKzhBeW5PVHVHazcxLwppZFpRU3hMbTRwUDVCdGZaQTB5dms0aGh5aEJ1VVRIY3B4dXVmV0gvT0JhS3dwelA3N041dnFKODQ2MWdFUlFUClBVazhLbGFWUlJ5TmJTNFlaVVMwbGNiWkpONVJ2L3E4WmtOaG5iNUI4Ti9RU3ZtK0NPNUMyRXJYV2xOOHpCbk8KN1ZSRzJtdXMrc1YxSGc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
        server: https://10.102.8.79:8443
      name: mk
    contexts:
    - context:
        cluster: mk
        user: system:node:mck-devops-tools
      name: system:node:mck-devops-tools@mk
    current-context: system:node:mck-devops-tools@mk
    kind: Config
    preferences: {}
    users:
    - name: system:node:mck-devops-tools
      user:
        client-certificate: cubulet-client-current.pem 
        client-key: cubulet-client-current.pem

и сертификат.

За основу взял из конфига в Задании 1.

Успешно подключаюсь к кластеру и вижу сервис

    C:\Users\Lenovo\.kube>
    C:\Users\Lenovo\.kube>kubectl get pods
    NAME                          READY   STATUS    RESTARTS   AGE
    hello-node-6b89d599b9-54fs2   1/1     Running   0          94m

    C:\Users\Lenovo\.kube>kubectl get services
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.110.194.131   <pending>     8080:31223/TCP   69m
    kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          36h

    C:\Users\Lenovo\.kube>


