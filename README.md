# Python Static and Reverse Proxy

Proxy written in Python. Included are a Static and a Reverse Proxy.

Static Proxy:
 * Proxy TCP- and Unix-sockets (Listen on both as well as proxy to both)
 * SSL encrypted sockets possible (via pyOpenSSL)

Reverse Proxy:
 * Modular design to build Reverse Proxies for many occasions (HTTP-Proxies based on Host-Header, Random-/Round-Robin-Proxies/LoadBalancers...)
 * Also TCP- and Unix-sockets supported
 * Also SSL encrypted sockets possible

## Table of Contents

   * [Python Static and Reverse Proxy](#python-static-and-reverse-proxy)
      * [Usage](#usage)
      * [Configure Reverse Proxy](#configure-reverse-proxy)
      * [Preconditions](#preconditions)
      * [Example](#example)

## Usage
 * For static proxy
  ```
usage: ./proxy.py staticproxy [-h] [--listen-port LISTEN_PORT]
                              [--listen-host LISTEN_HOST]
                              [--listen-unix LISTEN_UNIX]
                              [--listen-ssl LISTEN_SSL]
                              [--listen-ca-cert LISTEN_CA_CERT]
                              [--listen-cert LISTEN_CERT]
                              [--listen-key LISTEN_KEY]
                              [--proxy-port PROXY_PORT]
                              [--proxy-host PROXY_HOST]
                              [--proxy-unix PROXY_UNIX]
                              [--proxy-ssl PROXY_SSL]

optional arguments:
  -h, --help            show this help message and exit
  --listen-port LISTEN_PORT
                        Listen port for proxy only with --listen-host
  --listen-host LISTEN_HOST
                        Listen host for proxy only with --listen-port
  --listen-unix LISTEN_UNIX
                        Listen unix socket for proxy
  --listen-ssl LISTEN_SSL
                        Listen with SSL
  --listen-ca-cert LISTEN_CA_CERT
                        PEM-CA-Cert for listening
  --listen-cert LISTEN_CERT
                        PEM-Cert for listening
  --listen-key LISTEN_KEY
                        PEM-Key for listening
  --proxy-port PROXY_PORT
                        Port of the TCP proxy endpoint, only with --proxy-host
  --proxy-host PROXY_HOST
                        Host of the proxy endpoint, only with --proxy-port
  --proxy-unix PROXY_UNIX
                        Unix socket of the proxy endpoint
  --proxy-ssl PROXY_SSL
                        Proxy endpoint uses SSL?
  ```
 * For reverse proxy:

  ```
usage: ./proxy.py reverseproxy [-h] [--listen-port LISTEN_PORT]
                               [--listen-host LISTEN_HOST]
                               [--listen-unix LISTEN_UNIX]
                               [--listen-ssl LISTEN_SSL]
                               [--listen-ca-cert LISTEN_CA_CERT]
                               [--listen-cert LISTEN_CERT]
                               [--listen-key LISTEN_KEY]
                               [--proxy-module PROXY_MODULE]

optional arguments:
  -h, --help            show this help message and exit
  --listen-port LISTEN_PORT
                        Listen port for proxy only with --listen-host
  --listen-host LISTEN_HOST
                        Listen host for proxy only with --listen-port
  --listen-unix LISTEN_UNIX
                        Listen unix socket for proxy
  --listen-ssl LISTEN_SSL
                        Listen with SSL
  --listen-ca-cert LISTEN_CA_CERT
                        PEM-CA-Cert for listening
  --listen-cert LISTEN_CERT
                        PEM-Cert for listening
  --listen-key LISTEN_KEY
                        PEM-Key for listening
  --proxy-module PROXY_MODULE
                        Import .py file as a module containing a class named
                        ProxyHandler to handle reverse proxy functionality
  ```

## Configure Reverse Proxy

For reverse proxy functionality a module is required. This module contains callbacks, which are used to get proxy-address, modify data etc. Therefore this has to be self-written. To mention just a few circumstances, it's possbile to create Round-Robin-Proxies, insert HTTP X-Headers or request APIs to return the proxy-address.

 * Create a file with .py extension (xxx.py) containing a class named ProxyHandler.
 * Proxy handler must implement these methods:
```
class ProxyHandler(object):
    ### Initialize first end forward
    initial_handshake(self, socket)
        @socket: accepted socket
        @return: void

    ### Get Proxy Address from a file/an API ...
    get_reverse_proxy(self)
        @return: tuple|string, boolean -> (tuple tcp address|string unix address, needs_ssl?)

    ### Initialize second end forward
    init_reverse_proxy(self, socket)
        @socket: created proxy socket
        @return: boolean -> (False: init failed, True: init successfull)

    ### Modify data while forwarding
    @staticmethod
    modify_data(data)
        @data: data to forward
        @return: string -> (modified data)
```

 * Call ```./proxy.py reverseproxy --proxy-module=xxx.py --...```

## Preconditions

 * Python 2.7
 * PyOpenSSL

## Example

 * TCP-Proxy Docker-Socket: ```./proxy staticproxy --listen-host=0.0.0.0 --listen-port=4000 --proxy-unix=/var/run/docker.sock```
