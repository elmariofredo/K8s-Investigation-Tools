# Autoscaler Node to Node connection

I have run into autoscaler issue when about 30s after new worker node was marked as ready and pods start running on it, network connections were timing out with `Could not resolve host:` error message.

## Setup

In our setup we have two autoscaled node pools small and big with minimal 1 node.

## Debugging

1. Make sure you have cluster scaled to single node

2. Run service pod on first small worker. Update `nodeSelector` to match your small pool name.

```bash
➜ kubectl apply -f service-pod.yml
➜ kubectl get -f service-pod.yml -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE                                NOMINATED NODE   READINESS GATES
service-pod-big-agentpool   1/1     Running   0          10h   10.244.0.15   aks-agentpool-23867289-vmss000000   <none>           <none>
```

3. Update `1-pod-big.yml` and `1-pod-big.yml` with pod IP address of service-pod-big-agentpool pod ( run `kubectl get -f service-pod.yml -o wide` )

4. Run checking pod on first big pool worker to verify that connection is working ( check resource requirements if it will fit almost exactly to single big worker ).

```bash
➜ kubectl apply -f 1-pod-big.yml
➜ kubectl logs 1-pod-big -f
connected to 10.244.0.15:80 (237 bytes), seq=38173 time=  2.24 ms
connected to 10.244.0.15:80 (237 bytes), seq=38174 time=  2.81 ms
connected to 10.244.0.15:80 (237 bytes), seq=38175 time=  2.13 ms
connected to 10.244.0.15:80 (237 bytes), seq=38176 time=  2.33 ms
connected to 10.244.0.15:80 (237 bytes), seq=38177 time=  1.97 ms
connected to 10.244.0.15:80 (237 bytes), seq=38178 time=  1.94 ms
connected to 10.244.0.15:80 (237 bytes), seq=38179 time=  3.51 ms
connected to 10.244.0.15:80 (237 bytes), seq=38180 time=  2.34 ms
connected to 10.244.0.15:80 (237 bytes), seq=38181 time=  2.69 ms
...
```

5. Run testing pod to trigger big worker node autoscale and it is exactly the same as previous checking pod ( check resource requirements if it will fit almost exactly to single big worker ).

```bash
➜ kubectl apply -f 2-pod-big.yml
➜ kubectl get no # Wait for new worker node to get ready
➜ kubectl logs 2-pod-big -f
connect time out
connect time out
connect time out
connected to 10.244.0.15:80 (237 bytes), seq=38178 time=  1.94 ms
connected to 10.244.0.15:80 (237 bytes), seq=38179 time=  3.51 ms
connected to 10.244.0.15:80 (237 bytes), seq=38180 time=  2.34 ms
connected to 10.244.0.15:80 (237 bytes), seq=38181 time=  2.69 ms
...
```

