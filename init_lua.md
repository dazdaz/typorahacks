* remap 'Shift+Return' to 'Return'
* remap 'Return' to 'Shift+Return'
* https://www.hammerspoon.org/

```
-- global

hs.hotkey.bind({"cmd", "alt", "ctrl"}, "R", function()
  hs.reload()
end)
hs.alert.show("Config loaded")


local handleTyporaKeypress = function(event)
  local keyPressed = hs.keycodes.map[event:getKeyCode()]

  if keyPressed == 'return' then
    local flags = event:getFlags()

    if flags:containExactly({'shift'}) then
      -- handle shift + enter
      return true, {
        hs.eventtap.event.newKeyEvent({}, 'return', false), -- down
        hs.eventtap.event.newKeyEvent({}, 'return', true) -- up
      }
    elseif flags:containExactly({}) then
      -- handle enter only
      return true, {
        hs.eventtap.event.newKeyEvent({'shift'}, 'return', false), -- down
        hs.eventtap.event.newKeyEvent({'shift'}, 'return', true) -- up
      }
    end
  end

  return false
end

local typoraTap = hs.eventtap.new(
  { hs.eventtap.event.types.keyDown },
  handleTyporaKeypress
)

local function handleGlobalEvent(name, event, app)
  if event == hs.application.watcher.activated then
    local bundleId = string.lower(app:bundleID())

    if (bundleId:match("abnerworks.typora")) then
      -- start the keypress handler
      typoraTap:start()
    else
      -- stop the keypress handler, because you switched to another app
      typoraTap:stop()
    end
  end
end

watcher = hs.application.watcher.new(handleGlobalEvent)
watcher:start()
```
