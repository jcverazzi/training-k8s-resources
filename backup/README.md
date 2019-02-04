ETCDCTL_API=3

mkdir backup
cp -rf /etc/kubernetes/ssl backup

etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
    --cert=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1.pem \
    --key=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1-key.pem \
	member list

etcdctl --endpoints=https://127.0.0.1:2379     --cacert=/etc/kubernetes/ssl/etcd/ca.pem     --cert=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1.pem     --key=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1-key.pem     snapshot save backup/etcd-snapshot-latest.db

etcdctl --endpoints=https://127.0.0.1:2379     --cacert=/etc/kubernetes/ssl/etcd/ca.pem     --cert=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1.pem     --key=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1-key.pem     --write-out=table snapshot status backup/etcd-snapshot-latest.db

etcdctl --endpoints=https://127.0.0.1:2379     --cacert=/etc/kubernetes/ssl/etcd/ca.pem     --cert=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1.pem     --key=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1-key.pem  del "" --prefix
 
etcdctl --endpoints=https://127.0.0.1:2379     --cacert=/etc/kubernetes/ssl/etcd/ca.pem     --cert=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1.pem     --key=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user1-master-1-key.pem snapshot restore backup/etcd-snapshot-latest.db

etcd \
  --name m1 \
  --listen-client-urls http://localhost:2379 \
  --advertise-client-urls http://localhost:2379 \
  --listen-peer-urls http://localhost:2380 &

kubeadm config view > backup/admin.yaml

docker run --rm -v $(pwd)/backup:/backup \
    --network host \
    -v /etc/kubernetes/ssl/etcd:/etc/kubernetes/ssl/etcd \
    --env ETCDCTL_API=3 \
    k8s.gcr.io/etcd-amd64:3.2.18 \
    etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
    --cert=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user0-master-1.pem \
    --key=/etc/kubernetes/ssl/etcd/node-olea-form-k8s-user0-master-1-key.pem \
    snapshot save /backup/etcd-snapshot-latest.db

