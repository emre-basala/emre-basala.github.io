---
layout: post
title:  "Custom event handling in LUA and MOAI"
date:   2016-02-09 16:15:30
categories: lua moai
comments: true
---

<strong>Problem:</strong>

We want to trigger and event (or run a piece of code) when something else happens. How can we do this in LUA?

<strong>Solution:</strong>

We can store all the custom events and event handlers in a global hash table. Do a regular check running in a different thread to trigger events.


Let's say we want to show a `GameDialog` when a character finishes its walk. The implementation of the `GameDialog` is ommitted here because it's not relevant for the event handling mechanism.

Let's define `on_walk_finished` event handler

```
on_walk_finished = function (message)
  dialog_101 = GameDialog.new()
  GameDialog.run(dialog_101, "hi")
end
```

  It's a simple function to start a GameDialog.

  Let's define the `event` walk_finished.

```
walk_finished = function()
  x, y = Character.get_position(zeki)
  return distance(-100, x) < 10
end

```

Here the idea is to state "how do we know when the event has happened?". For simplicity, if the characters `X` value is close to `-100`, we can consider the walk is over.


<strong>How do we link those two methods?</strong>

We can define a EventManager class to check the registered events and trigger callbacks.

```
# event_manager.lua

EventManager = {
  set_timer = function(event, callback)
    game_env.events[event] = callback
  end,
}

function handle_events()
  for e,c in pairs(game_env.events) do
    if game_env.events[e] ~= nil then
      local result = e()
      if result then
        local callback = game_env.events[e]
        callback()
        game_env.events[e] = nil
      end
    end
  end
end

return EventManager
```


After requiring this class, we can use the `set_timer` method to link the `event` and the `event_handler`.


```
require "event_manager"

EventManager.set_timer(walk_finished, on_walk_finished)
```
After registring the events, we need to kick-start the event manager.

```

mainThread = MOAIThread.new ()
mainThread:run (
  function ()
    while true do
      coroutine.yield ()
      handle_events()
  end
end
)

```

Please let me know if this post was useful and whether there is a better solution to the problem.

