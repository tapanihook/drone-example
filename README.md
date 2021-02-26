# drone-example

Simple example to set up Drone for Github

## EC2

1. Launch an EC2 Linux. Assign Public IP

2. Create a shared secret
$ openssl rand -hex 16

3. Check EC2 instance Public IPv4 DNS

4. Create a GitHub OAuth app
- Profile / Settings / Developer settings / OAuth apps / New OAuth app 
  - Authorization callback URL: <public-ipv4-dns/login>
  - Note client id and secret

5. Run Drone Server


```
$ sudo docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID=<my-client-id> \
  --env=DRONE_GITHUB_CLIENT_SECRET=<my-client-secret> \
  --env=DRONE_RPC_SECRET=<my-shared-secret> \
  --env=DRONE_SERVER_HOST=<ec2-public-dns> \
  --env=DRONE_SERVER_PROTO=http \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1
```

6. Run Drone Runner

```
sudo docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=http \
  -e DRONE_RPC_HOST=<ec2-public-dns> \
  -e DRONE_RPC_SECRET=<my-shared-secret> \
  -e DRONE_RUNNER_CAPACITY=1 \
  -e DRONE_RUNNER_NAME=${HOSTNAME} \
  -p 3000:3000 \
  --restart always \
  --name runner \
  drone/drone-runner-docker:1
```
