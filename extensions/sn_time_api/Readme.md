# X4 Time API

This api provides additional real-time timing functions to mission director scripts. These timers will continue to run while the game is paused.

Timing is done using two sources:
* The lua function GetCurRealTime(), which measures the seconds since X4 booted up, advancing each frame while X4 is active (eg. not while minimized).
* Python 'time' module, accessed through the Named Pipes API, providing sub-frame timing.

Example uses:
- In-menu delays, eg. a blinking cursor.
- Timing for communication through a pipe with an external process.
- Code profiling.


### Requirements

* Optionally, Named Pipes API extension
  - https://github.com/bvbohnen/x4-named-pipes-api
  - Needed for the commands that utilize external python timing.


### Usage

General commands are sent using raise_lua_event of the form "Time.command".
Since multiple users may be accessing the timer during the same period,
each command will take an [id] unique string parameter.
Responses (if any) are captured using event_ui_triggered with screen "Time" and control [id].
Return values will be in "event.param3".

Warning: when a game is saved and reloaded, all active timers will be destroyed and any pending alarms will not go off.

Standard Commands:

- getEngineTime ([id])
  - Returns the current engine operation time in seconds, as a long float.
  - Note: this is the number of seconds since x4 was loaded, counting
    only time while the game has been active (eg. ignores time while
    minimized).
  - Capture the time using event_ui_triggered.
- startTimer ([id])
  - Starts a timer instance under id.
  - If the timer didn't exist, it is created.
- stopTimer ([id])
  - Stops a timer instance.
- getTimer ([id])
  - Returns the current time of the timer.
  - Accumulated between all Start/Stop periods, and since the last Start.
  - Capture the time using event_ui_triggered.
- resetTimer ([id])
  - Resets a timer to 0.
  - If the timer was started, it will keep running.
- printTimer ([id])
  - Prints the time on the timer to the debug log.
- setAlarm ([id]:[delay])
  - Sets an alarm to fire after a certain delay, in seconds.
  - Arguments are a concantenated string, colon separated.
  - Detect the alarm using event_ui_triggered.
  - Returns the realtime the alarm was set for, for convenience in
    creating clocks or similar.
  - Note: precision based on game framerate.

High precision commands (require Named Pipes API and running host server):

- getSystemTime ([id])
  - Returns the system time reported by python through a pipe.
  - Pipe communication will add some delay.
  - Can be used to measure real time passed, even when x4 is minimized.
- tic ([id])
  - Starts a fresh timer at time 0.
  - Intended as a convenient 1-shot timing solution.
- toc ([id])
  - Stops the timer associated with tic, returns the time measured,
    and prints the time to the debug log.


* Example: get engine time.
  ```xml
  <cue name="Test" instantiate="true">
    <conditions>
      <event_cue_signalled/>
    </conditions>
    <actions>
      <raise_lua_event name="'Time.getEngineTime'"  param="'my_test'"/>
    </actions>
  </cue>
  <cue name="Capture" instantiate="true">
    <conditions>
      <event_ui_triggered screen="'Time'" control="'my_test'" />
    </conditions>
    <actions>
      <set_value name="$time" exact="event.param3"/>
    </actions>
  </cue>
  ```
  
- Example: set an alarm.
  ```xml
  <cue name="Delay_5s" instantiate="true">
    <conditions>
      <event_cue_signalled/>
    </conditions>
    <actions>
      <raise_lua_event name="'Time.setAlarm'"  param="'my_alarm:5'" />
    </actions>
  </cue>  
  <cue name="Wakeup" instantiate="true">
    <conditions>
      <event_ui_triggered screen="'Time'" control="'my_alarm'" />
    </conditions>
    <actions>
      <.../>
    </actions>
  </cue>
  ```
