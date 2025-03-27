Some problems we encountered during the development:

1. OS error 61 in Sui Client CLI:
```
Networking or low-level protocol error: HTTP error: error trying to connect: tcp connect error: Connection refused (os error 61)

Caused by:
    HTTP error: error trying to connect: tcp connect error: Connection refused (os error 61)
```

This is because the RPC is not reachable. Any commands that start with `sui client` will not work in this case. 

This can happen if you previously set the network to a local network and now you are not running the local network. If so, you need to start the local network again in a new terminal by running `sui start --force-regenesis` (note that this will reset your local network). Keep this terminal running and then you can run the `sui client` commands in another terminal.