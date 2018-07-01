# ParticleMSG

A simple JSON messaging protocol for applications.

This protocol was designed with plugin systems in mind.

It should not be used for transmitting chunks of data. Use protocols like HTTP for that.

## Message structure

Every single message is JSON encoded and has `{"Type": ..., "Data": {...}}` form with exactly these field names and exactly this amount of fields.

Messagesare CRLF-separated (\r\n)

Message type is used to indicate purpose of the message while it's data field is used to transmit additional data.

Type field must be string, while data field must be an object.

## Connecting

ParticleMSG connection is established over a TLS socket.

It is recommended (but not necessary) for server to require clients sending their authentication keys.

Then you have to register your plugin/client like shown below:

## Registering

Client should send message displayed below in order to register itself:

```json
{"Type": "_register", "Data": {"Name": "pluginName", "Key": "a sha256-hashed auth key"}}
```

Then server responds with one of messages below:

```json
{"Type": "_alreadyRegistered", "Data": {"Name": "pluginName"}}
```

```json
{"Type": "_invalidKey", "Data": {"Key": "a sha256-hashed auth key"}}
```

```json
{"Type": "_blocked", "Data": {}} // means that server blocks everybody except for core
```

... and server API notifies user code with one of these messages:

```json
{"Type": "_alreadyRegistered", "Data": {"Who": "pluginName"}}
```

```json
{"Type": "_invalidKey", "Data": {"Who": "pluginName"}}
```

```json
{"Type": "_blocked", "Data": {"Who": "pluginName"}}
```

## Ping/Pong heartbeat

Every minute server sends next message to the client:

```json
{"Type": "_ping", "Data": {}}
```

And client must reply in another minute:

```json
{"Type": "_pong", "Data": {}}
```

If client fails to do so, client gets disconnected with next message:

```json
{"Type": "_quit", "Data": {"Reason": "Ping Timeout"}}
```

## Sending messages

Clients can send messages to other clients:

```json
core (write): {"Type": "_message", "Data": {"To": "chanop", "Message": {"Type": "ban", "Data": {"Who": "asdf", "Where": "##hjkl", "On": "asdfnet"}}}}
chanop (read): {"Type": "_message", "Data": {"From": "core", "Message": {"Type": "ban", "Data": {"Who": "asdf", "Where": "##hjkl", "On": "asdfnet"}}}}
```

## Subscribing and unsubscribing

Clients can subscribe to messages and unsubscribe from them.

```json
Client: {"Type": "_subscribe", "Data": {"ID": "aSubID", "Type": "msgType", "Pattern": {"Foo": "Bar"}, "Once": true}}
Server: {"Type": "_subscribed", "Data": {"ID": "aSubID"}}
```

In this way server can broadcast a message to everybody who has subscribed to a matching pattern.
For example, after sending message above, client will be able to receive broadcast-to-subscribed messages like this one:

```json
{"Type": "msgType", "Data": {"Foo": "Bar", "Baz": "Quux"}}
```

## Quitting

Client should send _quit message to disconnect properly:

```json
Client: {"Type": "_quit", "Data": {}}
Server: {"Type": "_quit", "Data": {"Reason": "Client Quit"}}
```