# node-red-contrib-mytimeout Readme
**Note:** Version changed to **2.1.1**. The code has changed the input from timer to timeout to match the documentation.

Was:

```
{
    "payload": "on",
    "timer": 3600,
    "warning": 300
}
```
is now

```
{
    "payload": "on",
    "timeout": 3600,
    "warning": 300
}
```

If the older message is sent, the timer will trigger but it will run with the default timer settings from the node and not the values passed in the JSON message.

If the node property 'Warning state payload' is not set or the property 'Warning (sec)' is set to 0, then no warning message will be sent.

If the timer is not currently running and a **stop** or **cancel** is sent to the timer, no output will be sent. An **on** will trigger the timer to begin running. If an **off** is sent while the timer is not running, an off message will be sent.

Also I've added a second output which sends out the time left on the timer.

```
{"payload":30,"state":1,"flag":"ticks > 0"}
or
{"payload":10,"state":2,"flag":"warn >= ticks"}
or
{"payload":0,"state":0,"flag":"off"}
or
{"payload":-1,"state":0,"flag":"stop"}
or
{"payload":-1,"state":0,"flag":"cancel"}
or
{"payload": -1, "state": 0, "flag": "unknown"}
```

When the **off** is sent or timeout occurs, the output 2 message is the same.
The **unknown** should never occur but is left in in case something breaks.

## myTimeout
MyTimeout started as hacked timer code I stole from Pete Scargill. The code would start a timer running if you sent it any trigger (tickle the timer). It would continue to run if further triggers were sent before the timeout occurred. A trigger was anything sent to the input of the timer.

My code goes some steps further. It allows you to tickle the timer, turn it off, stop it or cancel the timer. It also allows you to send JSON (see below) to override the timer, to turn off the timer or to cancel or stop the timer. It still retains Pete's behaviors. You can still send almost anything to tickle the timer with the exception of the JSON below which will cause the timer to do specific things.

You can configure the timeout module with the settings for the Safe and Unsafe messages. Safe is sent on the start of the timer, the unsafe message sent when the timeout occurs.

* Name
* Output topic
* On state payload
* Warning state payload
* Off state payload
* Countdown (sec)
* Warning (sec)

All of this is very useful for things like lights that are motion activated. If motion is detected, the Safe payload is sent (On or 1). The timer can be started when motion is detected and as long as the motion continues there will be no timeout (if motion occurs more often than the default timeout). When the timeout occurs the Unsafe payload is sent (Off or 0). When the timeout warning time (n seconds before the timeout) occurs the warning message is sent on output 1.

## MyTimeout
My additions to the code allows the user to send JSON in the triggers. You can override the timeout and warning times. 

```
{
    "payload": "on",
    "timeout": 3600,
    "warning": 300
}

{
    "payload": "off"
}

{
    "payload": "stop"
}

{
    "payload": "cancel"
}
```
### Payload
* **on** - turns on the timer. The addition fields *timeout* and *warning* are optional and allow the user to change the defaults. Both field values are integers and are in seconds. This payload will cause the timer to send a *Timer on payload* message in the msg.payload output
* **off** - turns off the timer. This payload will cause the timer to send a *Timer off payload* in the msg.payload output
* **stop** - stops the timer. This payload will cause the timer to send *stop* in the msg.payload output
* **cancel** - cancels the timer. This payload will cause the timer to cancel but send nothing in the msg.payload output
* If something other that the above (including no payload) it will be treated as a default **on** message. This allows messages to tickle the timer.

When sending the JSON **on** message, payload, timeout and warning are
all optional. When not present, the default node values will be
used. The JSON message **off** will immediately cause the timer node
to send an *Timer off payload* as the message and stop the timer. The
JSON **stop** message will stop the timer and send the stop as the
message. The JSON **cancel** message will stop the timeout and not
send a message. There is very little difference in the *off*, *stop*
and *cancel* behaviors other than what is sent or not sent to the
output 1. They all stop the timer.
