1. Elastic Container Register

# Create ECR
'''bash
aws ecr create-repository \
    --registry-id 492804330065 \
    --repository-name thuan-ecr/test-ecr \
    --profile PowerUserAccess-492804330065 \
    --region us-east-1
'''
# Delete ECR
'''bash
aws ecr delete-repository \
    --registry-id 492804330065 \
    --repository-name thuan-ecr/test-ecr \
    --profile PowerUserAccess-492804330065\
    --region us-east-1
'''
Login ECR
'''bash
# Private ECR
aws ecr-public get-login-password \
    --region us-east-1 \
    --profile PowerUserAccess-492804330065 \
| docker login \
    --username AWS \
    --password-stdin 492804330065.dkr.ecr.us-east-1.amazonaws.com
'''
# Build image
'''bash
#public ECR
docker tag thuan/vote:v1 public.ecr.aws/u3s1y1g0/thuan/thuan/vote:v1
docker build -t thuan-ecr/app .
docker tag thuan-ecr/app:latest 492804330065.dkr.ecr.us-east-1.amazonaws.com/thuan-ecr/test-ecr:v1
docker push 492804330065.dkr.ecr.us-east-1.amazonaws.com/thuan-ecr/test-ecr:v1
'''

2. Elastic Container Service
'''bash
#Create EKS
eksctl create cluster \
  --name thuan-cluster1 \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed \
  --spot

aws eks update-kubeconfig \
  --region us-east-1 \
  --name thuan-cluster1

# Create TLS key and certificates
openssl req -x509 -newkey rsa:4096 -days 365 -keyout ca-key.pem -out ca-cert.pem
base64 -w 0 cert/ca-cert.pem
base64 -w 0 cert/ca-key.pem
aws eks update-kubeconfig --region us-east-1 --name thuan-cluster1

# Install nginx-ingress-controller helm chart
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace test
helm uninstall ingress-nginx

