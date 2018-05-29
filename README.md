# Sockets - client layer

This is a jQuery application that uses WebSockets to communicate to our server via the [connection layer](https://github.com/dittto/socket-connection).

It waits for an active server and then shows the position of all clients, and allows the current client to change it's location.

To stop the WebSocket losing connection, this code uses [ReconnectingWebSocket()](https://github.com/joewalnes/reconnecting-websocket). This is handy for testing as you can restart the `connection layer` and have this code automatically reconnect to it.

## How to use it

The codebase is all HTML, CSS, and JavaScript, so you simply need somewhere to run it from.

If you're running all 3 layers of this Sockets demo locally, then you can simply drag the `index.html` in this project into *Google Chrome* and it'll start trying to connect over `ws://localhost:100`.

If you're running the `connection layer` remotely, you'll need to change the `ws://localhost:100` url in `index.html` to point at your remote server and port number.

## Deployment

To run this code remotely, first change that url to the location of your `connection layer`, such as `ws://<ec2-url>:8080`, and then upload this code. 

I'd recommend *AWS S3* for this as it's static code. All you need to do is import these files and folders (keeping the folder structure), and then change the S3 bucket permissions to allow static websites and to set all files in the bucket to be public.

## Testing

If you're just testing the `connection layer` and this code, you'll likely want to send some test commands to it.

Open up a new `http`-based window in *Google Chrome* and then open the *Developer tools* and from there, the *Console*.

Run the following commands to register a server and let the client know it's up:

```js
var conn = new WebSocket('ws://localhost:100');
conn.onopen = function (e) {
    console.log("Connection established!");
    conn.send('{"connect": "server"}');
    conn.send('{"data": {"server_active": true}}');
};
conn.onmessage = function (e) {
    console.log(e.data);
};
conn.onerror = function (e) {
    console.log("Error: " + e.data);    
};
```

The `"server_active": true` isn't anything special - it's simply some data. When a server is new, `data` is empty. We've added the `server_active` here to trick our client into starting when we want to.

If that connects correctly, you should see feedback that looks like the following, assuming you've already started the client:

```js
Connection established!
undefined
undefined
{"client_id":87,"client_data":[]}
```

You can ignore the undefined's, as that's the result of the `connection.send()` command. The important part is the id of the client in the next response.

Take that `client_id` and use it in the following command:

```js
conn.send('{"data": {"clients": [{"id": <client_id>, "x": 0.4, "y": 0.3}]}}');
```

Both the `x` and `y` positions are defined as between 0 and 1, to cope with different-size screens that can run a web-browser.

If you now look at the client, you'll see a green `o` showing in the location specified.

To test the client-to-server communication, click anywhere in the lighter-coloured box in the client. This will send a command like the following one to the server:

```js
{"client_id":<client_id>,"client_data":{"x":0.717741935483871,"y":0.8}}
```
