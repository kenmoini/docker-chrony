# Chronyd, in a Contaihner

> Self explanitory

```sh
# Build
docker build -t kenmoini/chrony .
# Run
docker run -it --rm --name chrony -p123:123/udp --cap-add SYS_TIME kenmoini/chrony
# Using the host's rtc (realtime clock) [untested]
docker run -it --rm --name chrony -p123:123/udp --cap-add SYS_TIME -v /dev/rtc:/dev/rtc:ro kenmoini/chrony
# BYO (bring your own) config file
docker run -it --rm --name chrony -p123:123/udp --cap-add SYS_TIME -v "$(pwd)"/chrony.conf:/etc/chrony.conf:ro kenmoini/chrony
# Run with arguments
docker run -it --rm --name chrony -p123:123/udp kenmoini/chrony -- --help
# Specify NTP servers to use as a time source
docker run -it --rm --name chrony -p123:123/udp --cap-add SYS_TIME kenmoini/chrony -s time.apple.com -s time.windows.com -s time.cloudflare.com
# Always restart / always online service and limit logfile size
docker run -d --restart always --name chrony -p123:123/udp --cap-add SYS_TIME --log-opt max-size=1m --log-opt max-file=3 kenmoini/chrony

# Save state and use host network
docker volume create chrony
docker run -d --restart always --name chrony --network host -v chrony:/var/lib/chrony --cap-add SYS_TIME kenmoini/chrony
```

```sh
# https://docs.docker.com/compose/compose-file/#cap_add-cap_drop
# this will not work :'( > Note: These options are ignored when deploying a stack in swarm mode with a (version 3) Compose file.
# https://github.com/docker/swarmkit/pull/1565
# docker stack deploy --compose-file=docker-compose.yml ntp-server
```

## Prevent conntrack from filling up

```sh
# get current status:
$ sysctl net.netfilter.nf_conntrack_count
$ sysctl net.netfilter.nf_conntrack_max

# disable conntrack on NTP port 123
$ iptables -t raw -A PREROUTING -p udp -m udp --dport 123 -j NOTRACK
$ iptables -t raw -A OUTPUT -p udp -m udp --sport 123 -j NOTRACK
