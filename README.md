# node-json-streamer
A really basic library wrapping a node TCP socket sending and parsing multiple JSON objects.
All this does is add an escape character on the client side (\u2603: A unicode snowman), split up the messages received on ]\u2603 || }\u2603, then emit 'msg' with the parsed JSON or nothing at all if it failed.
It does some error checking so if you send fluff down the socket then proper JSON it'll trim the fluff (without any kind of notification at all).
Oh it'll also randomly purge the buffer if it hits 512kb without any valid JSON. You know, just FYI.

## Installation
    $ npm link path/to/json-streamer

## Requirements
- A recent version of node. package.json says v0.6.0 but I'm sure it can handle less than that.
- Willingness to accept cuddles.

## Features
- Send and receive JSON reusing the same TCP connection
- Error correction, if you send fluff then valid JSON it will recover (by deleting the fluff)

## Example

```javascript
var jsonStreamer = require('jsonStreamer');

var server = jsonStreamer.listen(3001);

server.on('connection', function (client) {
  client.on('msg', function gotMsg(msg) {
    console.log('Server received message: ', msg.content);
    client.writeJSON({ ok: true });
  })
});

var client = jsonStreamer.connect(3001, function connectListener() {
  console.log("Connected");
  client.writeJSON({ a: 'b', c: 'd'});
  client.writeJSON(JSON.stringify({ b: 'b', d: 'd'}));
  client.writeJSON("BAHABABAHBAHABHA" + JSON.stringify({a: 'ok'}));
  client.writeJSON("OH GOD WHY");
  client.writeJSON('{"a":"b"}');
  client.writeJSON('{"a":');
  client.writeJSON('"b"}');
});

client.on('msg', function gotMsg(msg) {
  console.log('Client received message: ', msg.content);
});
```
