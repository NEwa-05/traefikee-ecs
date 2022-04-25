# Traefik Enterprise on ECS EC2

<!-- 
## add service discovery

Traefik is not working with service discovery since SRV dns record is mandatory for EC2 instances

```bash
aws servicediscovery create-private-dns-namespace --name traefikee --vpc vpc-069eaf8f964e322fe
aws servicediscovery get-operation --operation-id jv7lszczmxmlyvivmr74drn2upcjqxdc-jz3ah6yi
aws servicediscovery create-service --name controller --dns-config 'NamespaceId="ns-sdlqrfdwdxjpgjel",DnsRecords=[{Type="SRV",TTL="60"}]' --health-check-custom-config FailureThreshold=1
aws servicediscovery create-service --name registry --dns-config 'NamespaceId="ns-sdlqrfdwdxjpgjel",DnsRecords=[{Type="SRV",TTL="60"}]' --health-check-custom-config FailureThreshold=1
```
-->

## Set EC2 instance custom tag
```bash
aws ecs put-attributes --cluster ecs-ec2 --attributes name=controller,value=true,targetId=arn:aws:ecs:eu-north-1:${account-id}:container-instance/ecs-ec2/f2976ff74d7941be8b5cd1ee7a661d5e
aws ecs put-attributes --cluster ecs-ec2 --attributes name=registry,value=true,targetId=arn:aws:ecs:eu-north-1:${account-id}:container-instance/ecs-ec2/f2976ff74d7941be8b5cd1ee7a66172w
```

## Deploy TraefikEE Controller

### Register controller task definition
```bash
aws ecs register-task-definition --cli-input-json file://controller-task-ec2.json
```
### Create controller service
```bash
aws ecs create-service --cli-input-json file://controller-svc-ec2.json --cluster ecs-ec2 --enable-execute-command
```

### Execute command over controller container to get Proxy and Controller Tokens
```bash
aws ecs execute-command --cluster ecs-ec2 --container controller --task ${task_arn} --interactive --command "/traefikee tokens"
```

## Deploy TraefikEE Proxies

### Register proxies task definition
```bash
aws ecs register-task-definition --cli-input-json file://proxy-task-ec2.json
```

### Create Proxies service
```bash
aws ecs create-service --cli-input-json file://proxy-svc-ec2.json --cluster ecs-ec2
```

## Deploy TraefikEE Registry

### Register registry task definition
```bash
aws ecs register-task-definition --cli-input-json file://registry-task-ec2.json
```

### create registry service
```bash
aws ecs create-service --cli-input-json file://registry-svc-ec2.json --cluster dbl-ecs-ec2
```

## Remove TraefikEE

### delete ECS services and tasks
```bash
aws ecs delete-service --cluster dbl-ecs-ec2 --service traefikee-controller --force
aws ecs delete-service --cluster dbl-ecs-ec2 --service traefikee-proxies --force
aws ecs delete-service --cluster dbl-ecs-ec2 --service traefikee-registry --force
```

<!--
### delete Service Discovery services and namespace 
```bash
aws servicediscovery list-services
aws servicediscovery delete-service --id srv-yafh5w4yv3pyipi6
aws servicediscovery list-namespaces
aws servicediscovery delete-namespace --id ns-sdlqrfdwdxjpgjel
```
-->

### Delete ECS EC2 cluster
Drain instance, then delete instance and delete cluster
```bash
aws ecs delete-cluster --cluster ecs-ec2
```


