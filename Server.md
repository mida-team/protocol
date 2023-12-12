---
share: true
---
# Server
The format for a message is the following:
```json
{
	"t": <the message type>,
	"data": {
		<...>
	}
}
```

If batching is enabled, send an array of messages, along with the amount of the messages in the batch:
```json
{
	"count": 5,
	"data": [
		{
			"t": <...>
			"data": {
				<...>
			}
		},
		<the other 4 messages...>
	]
]
```

## Events

### Client Connection

On a client connection, you send the `0x00` event (server connection rules event). In the message sent by the server, it details:

* Heartbeat rates
* What permissions the client has (which is currently anonymous/unauthed)
* If batching is enabled
* idfk

The message looks like:

```json
{
	"t": 0,
	"data": {
		"hb": 500, // heartbeat rate in ms
		"perms": [
			// detailed on a later date
		],
		"batching": false
	}
}
```

### Server metadata

The client sends the `0x01` event (server metadata event). In the message send by the client, it details:

* Name of the server
* Approximation of online members
* Any other public server metadata

The message looks like:

```json
{
	"t": 1,
	"data": {
		"name": "Example Server",
		"online": 50,
		"uptime": 86400000,
		"maintenanceIn": 10800000,
		"note": "Have a wonderful day!",
		...
	}
}
```

### Auth 

The client sends the `0x02` event (Auth event) when it wishes to authenticate. Mida has no concept of passwords, and instead uses public keys to identify users. After authenticating, all other messages sent by the server must encrypted with the public key. Different servers can have different opinions on how public keys should work, but the recommended algorithm is [secp256k1](https://www.nervos.org/knowledge-base/secp256k1_a_key%20algorithm_(explainCKBot))

The message looks like this:

```json
{
	"t": 2,
	"data": {
		"pubKey": "0262b502019a7bb4f7185d4d067ddbc7faf0453c3cfa07030b89edbbcccc6f5923", // example public key
		... // space for additional information in the future
	}
}
```
## Example flow

| Sent by    | Event                |
| ---------- | -------------------- |
| ↗          | [Client Connection](Server.md#client%20-connection)    |
| ↙          | [Server metadata](Server.md#server%20-metadata)      |
| ↙          | [Auth](Server.md#auth)                 |

