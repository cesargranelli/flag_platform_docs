# Runbook

## Common Operations

### 1. Deploy Infrastructure

```bash
cd terraform/environments/dev
terraform init
terraform plan
terraform apply
```

### 2. Destroy Infrastructure

```bash
cd terraform/environments/dev
terraform destroy
```

### 3. SSH to Instance

```bash
ssh -i ~/.ssh/id_rsa ubuntu@<INSTANCE_IP>
```

### 4. Deploy Application

```bash
ansible-playbook -i '<INSTANCE_IP>,' \
  --extra-vars 'ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa' \
  ansible/playbooks/app-deploy.yml
```

### 5. View Logs

```bash
# Backend logs
docker logs -f spring-boot-backend

# Frontend logs
docker logs -f flutter-web
```

## Troubleshooting

### SSH Connection Timeout

1. Check security list rules in OCI Console
2. Verify instance is running
3. Check internet gateway and route table

### Container Not Starting

1. Check Docker logs: `docker logs <container_name>`
2. Verify environment variables
3. Check port conflicts

### Database Connection Issues

1. Verify database is available in OCI Console
2. Check connection strings
3. Verify network access (security lists)
