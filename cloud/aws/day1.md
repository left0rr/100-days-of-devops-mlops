# Day 1 — AWS EC2 Key Pair
## 🎯 Task

Create an RSA key pair named devops-kp in the us-east-1 region.

## 🧠 Concepts

- AWS CLI
- EC2 key pairs
- RSA keys
- Public vs private keys
- AWS regions

# 🔧 Commands
```bash
aws ec2 create-key-pair \
  --key-name devops-kp \
  --key-type rsa \
  --region us-east-1

aws ec2 describe-key-pairs \
  --key-names devops-kp \
  --region us-east-1