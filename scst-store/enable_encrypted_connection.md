# Enabling an Encrypted Connection to the Store

Depending on how your service's environment, you can enable an encrypted connection in different ways, either.

1. Using `LoadBalancer` 
1. Using `NodePort` — commonly used with local clusters such as kind, or minikube

After setting the connection, you'll get a CA certificate with which you can use to connect to the metadata store using the CLI or API.

## Using `LoadBalancer`

If you are using a `LoadBalancer` configuration, you need to first find the external IP of the `metadata-store-app` service. For all `kubectl` commands, make sure to use the `--namespace metadata-store` flag

```sh
$ kubectl get service/metadata-store-app --namespace metadata-store -o yaml
...
spec:
  ports:
  - name: http
    nodePort: 32712
    port: 8443
    protocol: TCP
    targetPort: 8443
...
status:
  loadBalancer:
    ingress:
    - ip: 10.186.124.220
```

In our example our IP is `10.186.124.220` and port is `8443`.

### Obtain the CA Certificate

The CA certificate is generated by cert manager. Run the following command to get the CA certificate:

```sh
$ kubectl get secret app-tls-cert -n metadata-store -o json | jq -r '.data."ca.crt"' | base64 -d > /tmp/ca.crt
```

Replace `/tmp/ca.crt` with where you want to save the CA certificate. This file will be used later when you [configure the CLI](configure_cli.md).

### Edit `/etc/hosts`

Add the IP entry mapping to `metadata-store-app.metadata-store.svc.cluster.local` in `/etc/hosts`:

```
10.186.124.220 metadata-store-app.metadata-store.svc.cluster.local
```

## Using `NodePort`

### Obtain the CA Cert

The CA Certificate is generated by cert manager. Run the following command to get the CA certificate:

```sh
$ kubectl get secret app-tls-cert -n metadata-store -o json | jq -r '.data."ca.crt"' | base64 -d > /tmp/ca.crt
```

Replace `/tmp/ca.crt` with where you want to save the CA certificate. This file will be used later when you [configure the CLI](configure_cli.md).

### Portforwarding
When using `NodePort`, in order for the CLI to access to the store, you need to start port forwarding for the service:

```sh
$ kubectl port-forward service/metadata-store-app 8443:8443 -n metadata-store
```

This command take over the terminal so your should run it in a separate terminal window.

### Modify your `/etc/hosts` file

Add the following entry to `/etc/hosts`:

```
127.0.0.1 metadata-store-app.metadata-store.svc.cluster.local
```
