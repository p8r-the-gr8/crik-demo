# crik-demo

### Steps

1) Spin up a registry instance:

```shell
docker run -d -p 5000:5000 --restart always --name registry registry:3
```

2) Spin up a cluster instance:

    1) Install Docker, Kind and Kubectl  
    2) Start Docker  
    3) Build the base image:  
    ```shell
    docker build -t crik-test:base -f Dockerfile.base .
    ```
    4) Build and push the looper image:  
    ```shell
    docker build -t <registry_instance_ip>:5000/crik-test:loop -f Dockerfile.loop .
    docker push <registry_instance_ip>:5000/crik-test:loop
    ```
    5) Adapt the Kind configuration (`cluster-config.yml`):   
    
        1) Replace <registry_instance_ip>
        2) Configure `/path/to/checkpoint/dir/on/host` and `/path/to/checkpoint/dir/in/cluster`
    6) Start cluster:  
    ```shell
    kind create cluster --config=cluster-config.yml
    ```
    7) Adapt the deployment configuration:  

        1) Replace <registry_instance_ip>
        2) Configure `/path/to/checkpoint/dir/in/cluster`
3) Peform the test:  
    1) Deploy:  
    ```shell
    kubectl apply -f cluster-config.yml
    ```
    2) Check the output:  
    ```shell
    kubectl logs $(kubectl get pods -o name) -f

    Command started with PID 9001
    Setting up SIGTERM handler to take checkpoint in /etc/checkpoint
    0
    1
    2
    3
    4
    5
    ...
    ```
    3) Delete the deployment:  
    ```shell
    kubectl delete -f cluster-config.yml
    ```
    4) Re-deploy:  
    ```shell
    kubectl apply -f cluster-config.yml
    ```
    5) Check the output:  
    ```shell
    kubectl logs $(kubectl get pods -o name) -f

    A checkpoint has been found in /etc/checkpoint. Restoring.
    125
    126
    127
    128
    129
    130
    131
    ...
    ```

    ### Shortcomings

    1) Currently, `crik` can only recover from a interruption:
    https://github.com/qawolf/crik/issues/4
    2) During testing, I'd found that it's best to delete the deployment before initiating an instance shutdowh/reboot to ensure the checkpoint is created.
    3) The main binary needs to be started by the `crik` binary. This is by design. To reduce developer pain, a base image can be made that uses `crik` for the `ENTRYPOINT` and other images can build on top of that using the `CMD` directive.