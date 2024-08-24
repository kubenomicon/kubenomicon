# etcd
While it's unlikely you'll need to directly interact with it, it's useful to know about the data store kubentes uses called etcd. Etcd is not a kubernetes specific technology, it's actually used by many other projects. According to the project site, etcd is:
> "A strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or cluster of machines. It gracefully handles leader elections during network partitions and can tolerate machine failure, even in the leader node."

At a high level, it's a version controlled key value store that can be interacted with using the `etcdctl` tool. The most basic example of creating a key/value pair is by running the command `etcdctl put hello world`. 

This very basic example simply maps the *key* of `hello` to the *value* of `world` which can then be queried by running `etcdctl get hello`

![](src/images/Pasted%20image%2020240325004034.png)

If we write this output to json using `etcdctl get --write-out=json hello | jq` we can see that there is actually a little bit more going on under the hood. First, the key/value pair are base64 encoded. Second we can see that there is a `create_revision` and a `mod_revision` number. Notice how this output is very simliar to `kubectl get <resource> -o json` command.

![](src/images/Pasted%20image%2020240325004338.png)

By updating the vlaue stored in `hello` to `hacker` using `etcdctl put hello hacker`, we can see that the value has been updated after querying the etcd server with `etcdctl get --write-out =json hello | jq`.  Interestingly, if we want to access the first value it stored with `etcdctl get --rev 8 --write-outjson hello | jq`, we can query etcd for the revision it was created at (in this case, 8) and it will give us our original `world`  value.


![](src/images/Pasted%20image%2020240325004538.png)

Etcd utilizes the [raft protocol](https://github.com/etcd-io/raft) to maintain state between nodes.