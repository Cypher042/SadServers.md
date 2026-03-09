# Lisbon

Description: There's an etcd server running on https://localhost:2379 , get the value for the key "foo", ie etcdctl get foo or curl https://localhost:2379/v2/keys/foo


```
admin@ip-10-1-12-83:/$ etcdctl get foo
Error:  client: etcd cluster is unavailable or misconfigured; error #0: x509: certificate has expired or is not yet valid: current time 2027-03-09T15:56:59Z is after 2023-01-30T00:02:48Z

error #0: x509: certificate has expired or is not yet valid: current time 2027-03-09T15:56:59Z is after 2023-01-30T00:02:48Z

admin@ip-10-1-12-83:/$ date
Tue Mar  9 15:57:33 UTC 2027

```

## Fixed the date and now the expired certificate error is gone. But

```
admin@ip-10-1-12-83:/$ etcdctl get foo
Error:  client: response is invalid json. The endpoint is probably not valid etcd cluster endpoint.
admin@ip-10-1-12-83:/$ curl https://localhost:2379/v2/keys/foo
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>

```
hmmm... Nginx???

on further analysis

```

admin@ip-10-1-12-83:/$ sudo ss -tulpn
Netid    State     Recv-Q    Send-Q                         Local Address:Port        Peer Address:Port    Process                                                                                                    
udp      UNCONN    0         0                                    0.0.0.0:68               0.0.0.0:*        users:(("dhclient",pid=409,fd=9))                                                                         
udp      UNCONN    0         0            [fe80::4bd:f4ff:fe13:c5bf]%ens5:546                 [::]:*        users:(("dhclient",pid=483,fd=8))                                                                         
tcp      LISTEN    0         4096                               127.0.0.1:2379             0.0.0.0:*        users:(("etcd",pid=576,fd=9))                                                                             
tcp      LISTEN    0         4096                               127.0.0.1:2380             0.0.0.0:*        users:(("etcd",pid=576,fd=8))                                                                             
tcp      LISTEN    0         128                                  0.0.0.0:22               0.0.0.0:*        users:(("sshd",pid=610,fd=3))                                                                             
tcp      LISTEN    0         511                                  0.0.0.0:443              0.0.0.0:*        users:(("nginx",pid=641,fd=6),("nginx",pid=640,fd=6),("nginx",pid=639,fd=6))                              
tcp      LISTEN    0         4096                                       *:6767                   *:*        users:(("sadagent",pid=569,fd=7))                                                                         
tcp      LISTEN    0         4096                                       *:8080                   *:*        users:(("gotty",pid=568,fd=6))                                                                            
tcp      LISTEN    0         128                                     [::]:22                  [::]:*        users:(("sshd",pid=610,fd=4))                                                                             
admin@ip-10-1-12-83:/$ 

```

Nginx... its running on port 443.... 

curl -v hows Nginx responding instead of etcd

why is port 443 responding when I am requesting to 2379.

I thought docker port mapping but nope its not it.

```

, options [nop,nop,TS val 2094558896 ecr 2094558896], length 287
00:01:06.893879 lo    In  IP localhost.2379 > localhost.46042: Flags [P.], seq 1782:2069, ack 709, win 1022, options [nop,nop,TS val 2094558896 ecr 2094558896], length 287
00:01:06.893924 lo    In  IP localhost.46042 > localhost.https: Flags [.], ack 2069, win 1024, options [nop,nop,TS val 2094558896 ecr 2094558896], length 0
00:01:06.893980 lo    In  IP localhost.2379 > localhost.46042: Flags [P.], seq 2069:2399, ack 709, win 1024, options [nop,nop,TS val 2094558896 ecr 2094558896], length 330
    
```

both 2379 and 443 (https)

are communicating when I am requesting to 2379 for some reason. This is not a nginx config thing cuz I am not even requesting at 443. 

After a lot of googling I found its something to do with those iptables

FOUND IT
```
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:2379 redir ports 443
DOCKER     all  --  anywhere            !ip-127-0-0-0.us-east-2.compute.internal/8  ADDRTYPE match dst-type LOCAL
```

Here deleting this ... finding this took a lot more time than I thought it would

Finallyyyy

```
admin@ip-10-1-12-166:/$ curl https://localhost:2379/v2/keys/foo
{"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":4,"createdIndex":4}}
admin@ip-10-1-12-166:/$ etcdctl get foo
bar
admin@ip-10-1-12-166:/$ 
```

GGs
