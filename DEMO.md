# Demo: Shopping Cart Conflict (Two Nodes)

This is a concise runbook for the two-node conflict demo using user-scoped iptables rules.

## Create Demo Users

```bash
sudo useradd -r -M nodea
sudo useradd -r -M nodeb

# Optional verification
getent passwd nodea
getent passwd nodeb
```

## Clone and Setup

```bash
git clone https://github.com/RAAHUL-tech/Mini_Dynamo /tmp/mini_dynamo_demo
chmod -R a+rx /tmp/mini_dynamo_demo
python3 -m venv /tmp/mini_dynamo_demo/.venv
/tmp/mini_dynamo_demo/.venv/bin/pip install -r /tmp/mini_dynamo_demo/requirements.txt
```

## Start Two Nodes

```bash
sudo -u nodea /tmp/mini_dynamo_demo/.venv/bin/python /tmp/mini_dynamo_demo/node.py --port 5001 --nodes 127.0.0.1:5001,127.0.0.1:5002
```

```bash
sudo -u nodeb /tmp/mini_dynamo_demo/.venv/bin/python /tmp/mini_dynamo_demo/node.py --port 5002 --nodes 127.0.0.1:5001,127.0.0.1:5002
```

## Before Partition


```bash
curl "http://localhost:5001/kv/cart?R=2&N=2"
```

```bash
curl -X PUT http://localhost:5001/kv/cart -H "Content-Type: application/json" -d '{"value":"[Apple]","N":2,"W":2}'
```

```bash
curl "http://localhost:5001/kv/cart?R=2&N=2"
```

## Partition Nodes

```bash
sudo iptables -A OUTPUT -p tcp -d 127.0.0.1 --dport 5002 -m owner --uid-owner nodea -j DROP
sudo iptables -A OUTPUT -p tcp -d 127.0.0.1 --dport 5001 -m owner --uid-owner nodeb -j DROP
```

## During Partition

```bash
curl "http://localhost:5001/kv/cart?R=1&N=2"
```

```bash
curl "http://localhost:5002/kv/cart?R=1&N=2"
```

```bash
curl -X PUT http://localhost:5001/kv/cart -H "Content-Type: application/json" -d '{"value":"[Apple,Banana]","N":2,"W":1}'
```

```bash
curl -X PUT http://localhost:5002/kv/cart -H "Content-Type: application/json" -d '{"value":"[Apple,Milk]","N":2,"W":1}'
```

```bash
curl "http://localhost:5001/kv/cart?R=1&N=2"
```

```bash
curl "http://localhost:5002/kv/cart?R=1&N=2"
```

## Heal the Network

```bash
sudo iptables -D OUTPUT -p tcp -d 127.0.0.1 --dport 5002 -m owner --uid-owner nodea -j DROP
sudo iptables -D OUTPUT -p tcp -d 127.0.0.1 --dport 5001 -m owner --uid-owner nodeb -j DROP
```

## After Heal

```bash
curl "http://localhost:5001/kv/cart?R=2&N=2"
```

## Cleanup

```bash
rm -rf /tmp/mini_dynamo_demo
sudo userdel nodea
sudo userdel nodeb
```
