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

7. Run Drone Runner(s)
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
