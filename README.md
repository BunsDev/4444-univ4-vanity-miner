# Uniswap V4 Vanity Miner

This repo contains three components to mine for the Uniswap V4 address:
- Modified version of [create2crunch](https://github.com/0age/create2crunch)
- Server instance to aggregate logs and forward salts to tg channel
- Script to bid on vastai instances

...

# Running

### Start The Log Aggregator Server

1. Setup a new tg bot + channel ([how to get the channel id](https://stackoverflow.com/questions/33858927/how-to-obtain-the-chat-id-of-a-private-telegram-channel))

2. Host server using railway
```bash
cd log-aggregator
railway init
railway up
```
> You will need to supply the env vars `BOT_TOKEN` and `CHANNEL_ID` on your railway dashboard


### Creating template on vastai

Head to [cloud.vast.ai/templates/edit](https://cloud.vast.ai/templates/edit). You can pull the docker-image [`mousless/fourfourfourfour`](https://hub.docker.com/repository/docker/mousless/fourfourfourfour/general) if you want to skip building your own.

Fill in the `on-startup Script` section with the following
```bash
for i in $(seq 0 $(($(clinfo | awk '/Platform Name/ {pname=$3} /Number of devices/ {if (pname ~ /NVIDIA/) {print $4}}') - 1))); do
  nohup /home/target/release/fourfourfourfour \
  0x48E516B34A1274f49457b9C6182097796D0498Cb \
  0x0000000000000000000000000000000000000000 \
  0x94d114296a5af85c1fd2dc039cdaa32f1ed4b0fe0868f02d888bfc91feb645d9 \
  $i \
  $YOUR_RAILWAY_ENDPOINT_URL_HERE \
  > "log_$i.txt" 2>&1 &
done
```
> Note that this is setting msg.sender as the zero address meaning anyone can submit your salt.

### Building Docker Image (Optional)

There is probably a much better way to mod create2crunch for this challenge. I just one shotted a prompt, it looked good, but it likely isn't the optimal solution.

```bash
cd create2crunch
docker build -t fourfourfourfour .
docker tag fourfourfourfour ${YOUR_DOCKERHUB_USERNAME}/fourfourfourfour:latest
docker push ${YOUR_DOCKERHUB_USERNAME}/fourfourfourfour:latest
```

### Setting up vastai bidder

vastai offers interruptable boxes where users can place bids on GPUs at a much lower price compared to straight out renting. This is a simple script to periodically place bids on RTX 4090s (up to $0.2/h)

Running bidder:
```bash
cd bidder
railway init
railway up
```
> You will need to supply the env vars `VAST_API_KEY` and `VAST_TEMPLATE_HASH` on your railway dashboard.