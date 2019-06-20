/usr/lib/node_modules/mediumexporter/index.js:32
  var s = json.payload.value;
                       ^

TypeError: Cannot read property 'value' of undefined
    at /usr/lib/node_modules/mediumexporter/index.js:32:24
    at Request._callback (/usr/lib/node_modules/mediumexporter/utils.js:13:16)
    at Request.self.callback (/usr/lib/node_modules/mediumexporter/node_modules/request/request.js:185:22)
    at Request.emit (events.js:197:13)
    at Request.<anonymous> (/usr/lib/node_modules/mediumexporter/node_modules/request/request.js:1161:10)
    at Request.emit (events.js:197:13)
    at IncomingMessage.<anonymous> (/usr/lib/node_modules/mediumexporter/node_modules/request/request.js:1083:12)
    at Object.onceWrapper (events.js:285:13)
    at IncomingMessage.emit (events.js:202:15)
    at endReadableNT (_stream_readable.js:1132:12)
    at processTicksAndRejections (internal/process/next_tick.js:76:17)
