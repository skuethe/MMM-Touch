## I. Install
```sh
cd <YOUR_MAGIC_MIRROR_DIRECTORY>/modules
git clone https://github.com/gfischershaw/MMM-Touch
```

## II. Configuration

### Simple
For testing or just for default predefined commands only.
```js
{
  module: "MMM-Touch",
  position: "top_right",
}
```

### Details & Defaults
> The following is the default configuration. Any and all values may be overridden in config/config.js.

```js
config: {
  debug: false, // When true creates a more detailed log.
  useDisplay: true, // When true displays the current command mode. Can be controlled at runtime via TOUCH_USE_DISPLAY.
  autoMode: false, // false or [] is disabled, or contains an array of:
                   // "module" == use module names
                   // "index" == use module indexes
                   // "instanceId" == use module instance ids
                   // anything == reference to any other mode names in the config
  threshold: {
    moment_ms: 1000 * 0.5, // TAP and SWIPE should be quicker than this.
    double_ms: 1000 * 0.75, // DOUBLE_TAP gap should be quicker than this. Set to zero to disable.
    press_ms: 1000 * 3, // PRESS should be longer than this.
    move_px: 50, // MOVE and SWIPE should go further than this.
    pinch_px: 50, // Average of traveling distance of each finger should be more than this for PINCH
    rotate_dg: 20, // Average rotating angle of each finger should be more than this for ROTATE
    idle_ms: 30000, // Idle time (in milliseconds) after which the defined "onIdle" notification / callback function should be triggered
  },
  defaultMode: "default", // default mode of commands sets on startup.This can also be an array. When set to an array backwards compatibility
                          // for getMode() is turned off, providing a consistent return type.
  onTouchStart: null, // special event on touch start - define a string for sending a custom notification or a function for a callback; f.e. onTouchStart: "TOUCH_ACTIVITY_NOTIFICATION"
  onTouchEnd: null, // special event on touch end - define a string for sending a custom notification or a function for a callback
  onIdle: null, // special event on "idle" - define a string for sending a custom notification or a function for a callback; see also: threshold.idle_ms for specifying the idle time
  gestureCommands: {}, // See next section.
  onNotification: () => {} // Special callback function for when you need something on notification received. Usually not needed.
}
```

### `onTouchStart` / `onTouchEnd`

When a touch event is started / ended, we are able to send a system wide notification or execute a custom function with these config option.  
Define a string for sending a notification. Define a function for doing what ever you want to. Some examples:

```js
// will send a TOUCH_ACTIVITY_STARTED notification
onTouchStart: "TOUCH_ACTIVITY_STARTED",

// will send a 500ms delayed MY_DELAYED_NOTI notification
onTouchEnd: () => {
  setTimeout(() => {
    this.sendNotification('MY_DELAYED_NOTI', ...)
  }, 500)
}
```

### `onIdle`

`onIdle` gives even more control. You can define a custom idle timeout via `config -> threshold -> idle_ms`.  
When no touch activitiy was recorded for that time, a notification (if set to `string`) or function will be executed. This is useful f.e. if you want to trigger some "screensaver" mode from another module.

```js
// will send a TOUCH_IDLE_TRIGGERED notification
onIdle: "TOUCH_IDLE_TRIGGERED",
```


### gestureCommands
A `mode` is typically a group of touch gestures all related to interacting with a type of module. For instance `newsfeed` responds to notifications to scroll back and forth in the feed, display a short article blurb, or the entire article itself, and so forth. Gestures can be defined for a mode which translate things like Swipe Left, Swipe Right, Double Tap, etc. into such notifications being sent. When the Touch module is set to that mode, either pro grammatically or by using `autoMode` (see below), those notifications will be sent.
```js
gestureComands: {
  "default": {
    ... // gestures under `default` mode
  },
  "some_mode": {
    ... // gestures under `some_mode` mode. For example, you can define different commands set for other situation.
  },
  ...
}

```
Each mode can have some definition about gesture and what to do on its triggering.
```js
gestureCommands: {
  "for_newsfeed": {
    "MOVE_LEFT_1": (commander, gesture) => {
      commander.sendNotification("ARTICLE_PREVIOUS")
    },
    "MOVE_RIGHT_1": (commander, gesture) => {
      commander.sendNotification("ARTICLE_NEXT")
    },
  },
  ...
}
```
With this example, if the current mode is `for_newsfeed`, an `ARTICLE_NEXT` notification is emitted when a single fingered move right touch gesture is detected. Each touch gesture is passed 2 arguments - `commander`, `gesture`. How to change the `mode`, and the `gesture` and `commander` arguments are detailed farther below.
### Default `gestureCommands`
```js
gestureCommands: {
  "default": {
    "TAP_1": (commander) => {
      commander.sendNotification("SHOW_ALERT", {
        title: "TOUCH Test.",
        timer: 3000,
      })
    },
    "PRESS_1": (commander) => {
      commander.getModules().forEach((m)=>{m.hide()})
      commander.setMode("hidden")
    }
  },
  "hidden": {
    "PRESS_1": (commander) => {
      commander.getModules().forEach((m)=>{m.show()})
      commander.setMode("default")
    }
  }
},
```
A `default` mode is predefined containing `PRESS_1` for toggling to hide/show modules and `TAP_1` for testing alert. As with all defaults it can be overridden.

### command
As seen above, each command has the structure:
```js
"GESTURE_FINGERS": callbackfunction (commander, gesture)
```
To define a custom command for tapping with 1 finger attach some actions to `TAP_1`:
```js
"TAP_1": (commander, gesture) => {
  console.log("tapping!")
  commander.sendNotification("SHOW_ALERT", {title:"I Tapped for " + gesture.duration + "sec.", timer:1000})
},
```

### argument : `commander`
`commander` is a helper class object to perform various actions in your command callback function:

- `commander.sendNotification(noti, payload)` : Emit any notifications with this method.
```js
commander.setNotification("SHOW_ALERT", {title: "test"})
```
- `commander.shellExec(shellCommand)` : Execute shell command for an external script or command with this method.
```js
commander.shellExec("sudo reboot now")
```
- `commander.getModule(moduleName | null)` : Retrieve any specified MM module object. If `null`, `MMM-Touch` will be returned.
```js
var module = commander.getModule("clock")
module.hide()
```
- `commander.getModules()` : Retrieve all MM modules.
```js
var modules = commander.getModules()
modules.forEach((module)=> {module.hide()})
```
- `commander.setMode(modeName)` : Change the current gestureCommands mode.
```js
commander.setMode("default")
```
- `commander.getMode()` : To get current mode.
```js
var currentMode = commander.getMode()
console.log(currentMode)
```
- `commander.forceCommand(mode, gestureObject)` : Generate a touch gesture in order to execute a command.
```js
commander.forceCommand("default", {gesture:"TAP", fingers:"1"})
```


### argument : `gesture`

Here is the structure of `gesture` object. When your gesture `TAP_1` is recognized, the `gesture` object will have properties similar to this.
```js
{
  fingers:1,
  gesture:"TAP",
  direction:"RIGHT",
  distance:0.45703125,
  duration:120.10000000009313,
  target: HTMLDOMObject, // Which DOM touch starts
  path: [HTMLDOMObject, HTMLDOMObject, ...],  // Deep HTML DOM Tree of target
  //pinchSum : 123.456, // on `PINCH`, this value will be included. This means the sum of distance of pinching fingers
  //degree : 23.45, // on `ROTATE`, this value will be included. This means the average degree of rotation of each finger.
  instanceId: "nytimes", // If autoMode is enabled and if the module instance that receives a touch event has defined an "instanceId" then it will be passed through. Otherwise undefined. This should be passed as part of the payload to sendNotification() for modules that can support multiple instances. When calling forceCommand() the instanceId can be passed in the payload.
}
```
Typically most elements of this object can be ignored. It can however be used to gain additional details, such as the velocity of the gesture.


### onNotification
In order to respond to notifications from modules, such as changing the current `mode`, use `onNotification`:
```js
onNotification: (commander, notification, payload, sender)=> {
  if (notification == "SPOTIFY_UPDATE_PLAYING") {
    if (payload == true) commander.setMode("mode_spotify")
    if (payload == false && commander.getMode() == "mode_spotify") commander.setMode("default")
  }
  if (notificaiton == "PAGE_CHANGED" && payload == 2) {
    commander.setMode("page2")
  }
  ...
}
```

## III. Notification
Other modules can control this module with `notification`.

### **`TOUCH_USE_DISPLAY`** 
- `payload:true|false` : To show/hide detected gestures on MM screen.
```js
this.sendNotification("TOUCH_USE_DISPLAY", false)
```

### **`TOUCH_GET_MODE`**
- `payload: (mode)=>{}` : To get current mode with callback function.
```js
this.sendNotification("TOUCH_GET_MODE", (mode) => {
  console.log(mode)
})
```

### **`TOUCH_SET_MODE`**
- `payload: String` : To set mode
```js
this.sendNotification("TOUCH_SET_MODE, "default")
```

### **`TOUCH_REGISTER_COMMAND`**
- `payload: {mode, gesture, callback}` : To register new command on runtime dynamically
```js
this.sendNotification("TOUCH_REGISTER_COMMAND", {
  mode: "default",
  gesture: "PRESS_2",
  func: (commander) => {
    commander.shellExec("sudo shutdown now")
  }
})
```

### **`TOUCH_FORCE_COMMAND`**
- `payload: {mode, gestureObject}` : To execute command by force 
```js
this.sendNotification("TOUCH_FORCE_COMMAND", {
  mode: "default",
  gestureObject: {
    gesture: "TAP",
    fingers: "1",
    instanceId: "nytimes", // optional
  }
}
```

### **`TOUCH_CANCEL`**
To cancel the current gesture (new gesture detection will be started after all fingers have been released).
```js
this.sendNotification("TOUCH_CANCEL")
```

## IV. Gestures
### `TAP` and `PRESS`
- **`TAP`** : If the time from the first touch to the last release is less than 0.5s (default defined in configuration) and no finger travels more than a minimum defined distance, the `TAP` event will occur.
- **`DOUBLE_TAP`** : If the time between two single finger `TAP` gestures is less than 0.75s then the second `TAP` is replaced by a `DOUBLE_TAP`.
- **`PRESS`** : When you are pressing fingers for 3s (defined in configuration), even without releasing, the `PRESS` will occur. (like `power` button of electronic appliances). If you want to make `PRESS` event again, release fingers then do the gesture again. (not continuously)

### `SWIPE` and `MOVE`
- **`SWIPE`** : Similar to `TAP`, but fingers are traveled and released in some distances (>50px) and time (<0.5s), `SWIPE` event will occur.
- **`MOVE`** : Unlike `SWIPE`, Without releasing, `MOVE` events could occur. it is continuously. So, you can activate gesture `LEFT-LEFT-LEFT...` without finger releasing. It would be useful when you need continuous control like changing volume seamlessly.

### `PINCH` and `ROTATE`
- **`PINCH`** : Similar to `MOVE`. This is continuously recognizable. The average distance of traveling of each finger from centroid should be over the threshold value.
- **`ROTATE`** : The same. Your 2 fingers would make a `vector`. The angle(degree) between your `start vector` and `end vector` should be over the threshold value. This is continuously recognizable without releasing also.



## V. Possible issues
> If you have some troubles, first set `debug:true` to get detail information. (In your front-end dev-console.)

### `Tap` vs `Click`, event bubbling
If finger-touch starts on some element that has already `onclick` event, Touch gesture will not starts. However, JS and HTML DOM are very sophisticated, there could be a possibility of unrecognition on complex DOM trees. If possible, to start touching on the empty area is recommended.

There could be another possibility of event bubbling besides `onclick`. If happens, report me.

### DOM refreshing issue
Similar to the previous issue. If the `DOM`(module or element) where your touch starts is updated/removed during the movement, Gesture might be lost. So, start touching in a safe area (e.g: empty space).

### Rotate?
I'm not good at Math. so my algorithm for determining `rotation` might be wrong. I can't calculate more than 2 fingers rotation. If anyone can deal, feel free to make PR.



