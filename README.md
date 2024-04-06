# pluto-websocket

WebSocket client library for Pluto.

## Documentation

This library extends Pluto's `socket` library, adding the following functions:

- `socket.wsUpgrade(sock, host, path = "/")`
- `socket.wsSend(sock, payload, binary = false)`
- `socket.wsRecv(sock)`

## Example Code

```lua
-- Require Pluto's built-in socket library
local socket = require "pluto:socket"

-- Require WebSocket library to add more functions to 'socket' library
require "websocket"

local host <const> = "ws.postman-echo.com"
local sock = socket.connect(host, 443)
if sock:starttls(host) and sock:wsUpgrade(host, "/raw") then
    print "Upgraded to WS over TLS"
    sock:wsSend("Hello from Pluto!")
    print("Got echo back: "..sock:wsRecv())
end
```
