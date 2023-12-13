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

You should also include information such as whether the user is signing up or logging in. Information relating to each method should be placed accordingly.

The message looks something like this:

```json
{
	"t": 2,
	"data": {
		"pubKey": "0262b502019a7bb4f7185d4d067ddbc7faf0453c3cfa07030b89edbbcccc6f5923", // example public key
		"method": "signup",
		"user": {
			"displayName": "John Doe",
			"bio": "\"Remember switching to your secondary is always faster than reloading\""
		}
		... // space for additional information in the future
	}
}
```

### Auth Challenge

After the client sends the [Auth](Server.md#Auth) event, the server sends the `0x03` (Auth Challenge event). It doesn't matter if the authentication was successful at this point, but this is used to make sure that the user is actually the owner of the account. The challenge can be really anything, but we recommend creating it like this:

```ts
const values = crypto.getRandomValues(new Uint8Array(64));
const encodedValues = encodeHex(values);
const challenge = sha256(encodedValues);
```

The message looks something like this:

```json
{
	"t": 3,
	"data": {
		"challenge": "04dfa7e237c000d849dc11f01aac01bdcffe7f603e93b60e7c9e85cca3ddd4e3"
	}
}
```

### Auth Challenge Solution

The client sends the `0x04` event (Auth Challenge Solution event) to solve the challenge event. What the client does is take the challenge, and then sign it with its private key. The message looks like:

```json
{
	"t": 4,
	"data": {
		"solution": "621066c77d17d49914ee4de550cf09dc11d5568bfb6e56cea3c457936ca80fd14825dc9e2ed3757afc7fbc52e4f49e2017c58606e4bdcbff77058ee083b01c97"
	}
}
```

### Auth Result

The server sends the `0x05` event (Auth Result event) after the client attempts to solve the challenge event. You verify their signature with their public key, and if it passes, you send a message like:

```json
{
	"t": 5,
	"data": {
		"passed": true,
		"user": {
			"displayName": "John Doe",
			"unreadMessages": 4
		}
	}
}
```

If the client does not pass, then you send a message like:

```json
{
	"t": 5,
	"data": {
		"passed": false
	}
}
```

You can accept another attempt at the challenge from the client. We recommend accepted 2 more (3 in total) attempts from the client in the case a technical error or malformed message. If they fail those as well, you can then close the connection.
## Example flow

| Sent by    | Event                   |
| ---------- | ----------------------- |
| ↗          | [Client Connection](Server.md#client-connection)       |
| ↙          | [Server metadata](Server.md#server-metadata)         |
| ↙          | [Auth](Server.md#auth)                    |
| ↗          | [Auth Challenge](Server.md#Auth-Challenge)          |
| ↙          | [Auth Challenge Solution](Server.md#Auth-Challenge-Solution) |
| ↗          | [Auth Result](Server.md#Auth-Result)             |



