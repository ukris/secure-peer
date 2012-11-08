# secure-peer

Create encrypted peer-to-peer streams using public key cryptography.

No certificates, no authorities. Each side of the connection has the same kind
of keys so it doesn't matter which side initiates the connection.

# example

First generate some public/private keypairs with
[rsa-json](http://github.com/substack/rsa-json):

```
$ rsa-json > a.json
$ rsa-json > b.json
```

``` js
var secure = require('secure-peer');
var peer = secure(require('./a.json'));

var net = require('net');
var server = net.createServer(function (rawStream) {
    var sec = peer(function (stream) {
        stream.pipe(stream); // simple echo server
    });
    sec.pipe(rawStream).pipe(sec);
});
server.listen(5000);
```

``` js
var secure = require('secure-peer');
var peer = secure(require('./b.json'));

var net = require('net');
var rawStream = net.connect(5000);

var sec = peer(function (stream) {
    stream.pipe(process.stdout);
    stream.write('beep boop\n');
});
sec.pipe(rawStream).pipe(sec);

sec.on('identify', function (id) {
    // you can asynchronously verify that the id.key is known here
    id.accept();
});
```

For extra security, you should keep a file around with known hosts to verify
that the public key you receive on the first connection doesn't change later
on like how `~/.ssh/known_hosts` works.

Maintaining a known hosts file is outside the scope of this module.

# methods

``` js
var secure = require('secure-peer')
```

## var peer = secure(keys)

Return a function to create streams given the `keys` supplied.

`keys.private` should be a private PEM string and `keys.public` should be a
public PEM string.

You can generate keypairs with [rsa-json](http://github.com/substack/rsa-json).

## var sec = peer(cb)

Create a new duplex stream `sec`

# events

## sec.on('connection', function (stream) {})

Emitted when the secure connection has been established successfully.

`stream.id` is the identify object from the `'identify'` event.

## sec.on('identify', function (id) {})

Emitted when the connection identifies with its public key, `id.key`.

Each listener *must* call either `id.accept()` or `id.reject()`.

The connection won't be accepted until all listeners call `id.accept()`. If any
listener calls `id.reject()`, the connection will be aborted.

### id.accept()

Accept the connection. This function must be called for every listener on the
`'identify'` event for the connection to succeed.

### id.reject()

Reject the connection. The connection will not succeed even if `id.accept()` was
called in another listener.

## sec.on('header', function (header) {})

Emitted when the remote side provides a signed header.payload json string signed
with its private key in header.hash.

# install

With [npm](https://npmjs.org) do:

```
npm install secure-peer
```

# license

MIT