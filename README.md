# drone-example

Simple example to set up Drone for Github. 

## AWS EC2

Note: For this example you need a domain in Route 53. TLS is used

1. Launch Amazon Linux instance and install Docker. Assign Public IP to instance.
$ sudo yum update -y
$ sudo yum install docker -y
$ sudo service docker start
$ sudo usermod -a -G docker ec2-user

2. Assign Public IP to instance
3. Create an a-record pointing to the instance. Remember to check security group rules
4. Create a shared secret
$ openssl rand -hex 16
5. Create a GitHub OAuth app
- Profile / Settings / Developer settings / OAuth apps / New OAuth app 
  - Authorization callback URL: <my-domain/login>
  - Note client id and secret
6. Run Drone Server
```
$ sudo docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID=<my-client-id> \
  --env=DRONE_GITHUB_CLIENT_SECRET=<my-client-secret> \
  --env=DRONE_RPC_SECRET=<my-shared-secret> \
  --env=DRONE_SERVER_HOST=<my-domain> \
  --env=DRONE_SERVER_PROTO=https \
  --env=DRONE_TLS_AUTOCERT=true \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1
```

7. (Optional) Run Drone Runner(s)
```
sudo docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=https \
  -e DRONE_RPC_HOST=<my-domain> \
  -e DRONE_RPC_SECRET=<my-shared-secret> \
  -e DRONE_RUNNER_CAPACITY=1 \
  -e DRONE_RUNNER_NAME=${HOSTNAME} \
  -p 3000:3000 \
  --restart always \
  --name runner \
  drone/drone-runner-docker:1
```

8. (Optional) Run Drone AutoScaler. Launches instance(s) if there are pending builds
```
sudo docker run -d \
  -v /var/lib/autoscaler:/data \
  -e DRONE_POOL_MIN=1 \
  -e DRONE_POOL_MAX=2 \
  -e DRONE_SERVER_PROTO=https \
  -e DRONE_SERVER_HOST=<my-domain> \
  -e DRONE_SERVER_TOKEN=<my-drone-personal-token> \
  -e DRONE_AGENT_TOKEN=<my-shared-secret> \
  -e DRONE_AMAZON_INSTANCE=t3.medium \
  -e DRONE_AMAZON_REGION=eu-west-1 \
  -e DRONE_AMAZON_SUBNET_ID=<aws-subnet-id> \
  -e DRONE_AMAZON_SECURITY_GROUP=<aws-security-group-id> \
  -e DRONE_AMAZON_SSHKEY=<my-aws-keypair> \
  -e DRONE_LOGS_DEBUG=true \
  -e AWS_ACCESS_KEY_ID=<> \
  -e AWS_SECRET_ACCESS_KEY=<> \
  -p 8080:8080 \
  --restart=always \
  --name=autoscaler \
  drone/autoscaler
```
