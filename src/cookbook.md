# Cookbook

## 1. Timed session

This automatically ends the session after 3 seconds

```python
from pyControl.utility import *
from devices import *

states = ["state_a"]
events = ["session_over"]

v.session_duration = 3 * second

initial_state = "state_a"


def run_start():
    set_timer("session_over", v.session_duration)


def state_a(event):
    if event == "session_over":
        stop_framework()
```

## 2. Blinking light

Demonstrates how to make multiple LEDs blink at different frequencies asynchronously.
In this task, the red and green LEDs alternate: each blinks for 2 seconds before switching, while the blue LED blinks continuously at its own frequency.

```python
from pyControl.utility import *
from devices import *


# hardware
breakout = Breakout_1_2()
blue_LED = Digital_output(breakout.LED_blue)
red_LED = Digital_output(breakout.LED_red)
green_LED = Digital_output(breakout.LED_green)

# states and events
states = ["red_state", "green_state"]
events = ["toggle_red", "toggle_green", "toggle_blue"]

# variables
v.red_frequency = 5 # Hz
v.green_frequency = 10 # Hz
v.blue_frequency = 3 # Hz

initial_state = "red_state"


def run_start():
    set_timer("toggle_blue", 1000.0 / v.blue_frequency)


def all_states(event):
    if event == "toggle_blue":
        blue_LED.toggle()
        set_timer("toggle_blue", 1000.0 / v.blue_frequency)


def red_state(event):
    if event == "entry":
        timed_goto_state("green_state", 2 * second)
        set_timer("toggle_red", 1000.0 / v.red_frequency)
    elif event == "toggle_red":
        red_LED.toggle()
        set_timer("toggle_red", 1000.0 / v.red_frequency)
    elif event == "green_state":
        goto_state("green_state")
    elif event == "exit":
        red_LED.off()


def green_state(event):
    if event == "entry":
        timed_goto_state("red_state", 2 * second)
        set_timer("toggle_green", 1000.0 / v.green_frequency)
    elif event == "toggle_green":
        green_LED.toggle()
        set_timer("toggle_green", 1000.0 / v.green_frequency)
    elif event == "exit":
        green_LED.off()

```

## 3. Button hold threshold

Demonstrates how to detect when a button is held for a certain duration.

```python
from pyControl.utility import *
from devices import *

breakout = Breakout_1_2()
button = Digital_input(
    breakout.button,
    pull="up",
    falling_event="button_press",
    rising_event="button_release",
)

states = ["wait_for_hold"]
events = ["button_press", "button_release", "hold_complete"]

v.hold_time = 2 * second

initial_state = "wait_for_hold"


def wait_for_hold(event):
    if event == "button_press":
        set_timer("hold_complete", v.hold_time)
    elif event == "button_release":
        ms_short = timer_remaining("hold_complete")
        disarm_timer("hold_complete")
        if ms_short > 0:
            print("Button not held long enough, {} ms remaining".format(ms_short))
    elif event == "hold_complete":
        print("Button successfully held for {} ms".format(v.hold_time))
```

<!-- ## 4. N button press threshold

```python

```

## 5. Analog triggers

## 6. Printing

Reccommend upgrading to Micropython v1.12 or above to take advantage of f-strings -->