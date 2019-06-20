/usr/lib/node_modules/mediumexporter/utils.js:32
      var tokens = res.body.match(/youtube.com%2Fembed%2F([^%]+)%3F/);
                       ^

TypeError: Cannot read property 'body' of undefined
    at Request._callback (/usr/lib/node_modules/mediumexporter/utils.js:32:24)
    at self.callback (/usr/lib/node_modules/mediumexporter/node_modules/request/request.js:185:22)
    at Request.emit (events.js:197:13)
    at Request.onRequestError (/usr/lib/node_modules/mediumexporter/node_modules/request/request.js:881:8)
    at ClientRequest.emit (events.js:197:13)
    at TLSSocket.socketErrorListener (_http_client.js:397:9)
    at TLSSocket.emit (events.js:197:13)
    at emitErrorNT (internal/streams/destroy.js:82:8)
    at emitErrorAndCloseNT (internal/streams/destroy.js:50:3)
    at processTicksAndRejections (internal/process/next_tick.js:76:17)
