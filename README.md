# BankPro Core (Multi-Cloud Demo)

A minimal Spring Boot service deployed via Jenkins + Ansible + Docker to AWS and Azure VMs.

## Endpoints
- `/api/hello`

## Inventory (current)
- AWS: 54.90.191.189
- Azure: 130.131.219.38

## Build locally (optional)
```
mvn clean package
docker build -t docker.io/jhimanshu50/bankpro-core:latest .
```
