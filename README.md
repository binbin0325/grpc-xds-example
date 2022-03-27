## Setup

edit `/etc/hosts`

```bash
127.0.0.1 be.cluster.local xds.domain.com
```

## gRPC Server

Start gRPC Servers

```bash
cd app/
go run src/server/grpc_server.go --grpcport :50051 --servername server1
```

> **NOTE** If you change the ports or run more servers, ensure you update the list when you start the xDS server

## xDS Server

Now start the xDS server:

```bash
cd xds-new
go run main.go 
```

> **NOTE** Update the list of `--upstream_port` flags to reflect the list of ports for the gRPC servers that you started


## gRPC Client (xDS)
- `xds_bootstrap.json`:
```json
{
  "xds_servers": [
    {
      "server_uri": "xds.domain.com:18000",
      "channel_creds": [
        {
          "type": "insecure"
        }
      ],
      "server_features": ["xds_v3"]     
    }
  ],
  "node": {
    "id": "b7f9c818-fb46-43ca-8662-d3bdbcf7ec18~10.0.0.1",
    "metadata": {
      "R_GCP_PROJECT_NUMBER": "123456789012"
    },
    "locality": {
      "zone": "us-central1-a"
    }
  }
}
```

Then:

```bash
go run src/client/grpc_client.go
```

in the debug logs that it connected to port `:50051`

```console
INFO: 2020/04/21 16:14:42 Subchannel picks a new address "be.cluster.local:50051" to connect
```

The grpc client will issue grpc requests every 5 seconds using the list of backend services it gets from the xds server.
Since the xds server will abruptly rotate the grpc backend servers, the client will suddenly connect to `server2`

```console
INFO: 2020/04/21 16:16:08 Subchannel picks a new address "be.cluster.local:50052" to connect
```

The port it connected to is `:50052`

Right, that's it!

---

```log
$ go run src/grpc_client.go 

2021/06/15 10:47:26 RPC Response: 0 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:47:31 RPC Response: 1 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:47:36 RPC Response: 2 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:47:41 RPC Response: 3 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:47:46 RPC Response: 4 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:47:51 RPC Response: 5 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:47:56 RPC Response: 6 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:48:01 RPC Response: 7 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:48:06 RPC Response: 8 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:48:11 RPC Response: 9 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:48:16 RPC Response: 10 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:48:21 RPC Response: 11 message:"Hello unary RPC msg   from server1" 
2021/06/15 10:48:26 RPC Response: 12 message:"Hello unary RPC msg   from server2"  
2021/06/15 10:48:31 RPC Response: 13 message:"Hello unary RPC msg   from server2" 
2021/06/15 10:48:36 RPC Response: 14 message:"Hello unary RPC msg   from server2" 
```

---

If you want more details...

## References

- [Envoy Listener proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener.proto)
- [Envoy Cluster proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cluster.proto)
- [Envoy Endpoint proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint.proto)

