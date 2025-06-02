# pluto-websocket

WebSocket client & server library for Pluto.

## Documentation

This library adds the following functions to Pluto's `socket` library:
- `socket.wsUpgrade(sock, ...)`
    - For client sockets: `socket.wsUpgrade(sock, host, path = "/")` 
    - For server sockets: `socket.wsUpgrade(sock, key)` 
- `socket.wsSend(sock, payload, binary = false)`
- `socket.wsRecv(sock)`

and exports the following by itself:
- `wsIsUpgradeRequest(text)` — returns nil or a key to be used with wsUpgrade

## Example Code

Client-only usage with TLS:

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

Client & server usage:
```lua
local { scheduler, socket } = require "*"
local { wsIsUpgradeRequest } = require "websocket"

local sched = new scheduler()

-- Server
socket.bind(sched, 8080, function(s)
    if req := s:recv() then
        if key := wsIsUpgradeRequest(req) then
            print("Server: Accepting a websocket upgrade.")
            s:wsUpgrade(key)
            while data := s:wsRecv() do
                print("Server: Got frame: "..data)
                s:wsSend(data)
            end
        end
    end
    s:close()
end)

-- Client
sched:add(function()
    local sock = socket.connect("localhost", 8080)
    if sock:wsUpgrade("localhost", "/") then
        print "Client: Upgraded to websocket."
        sock:wsSend("Hello from Pluto!")
        print("Client: Got echo back: "..sock:wsRecv())
    end
end)

sched:run()
```
