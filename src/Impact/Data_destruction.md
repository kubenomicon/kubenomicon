# Data Destruction
The impact of data destruction is fairly obvious. If an attacker has the ability to destroy data, they may be able to delete information from the cluster. This can be both to cause denial of service, or to remove unbacked up data from a cluster.

The following command is the functionality equivalent to running `rm -rf --no-preserve-root /` on a normal Linux machine: `kubectl delete all --all --all-namespaces <remove_this_to_run> --grace-period=0 --force`