# Tutorial

## 1. Bare minimum
At a minimum, a task must define a list of states and events, and the initial state.

This task doesn't do anything. When the task begins, it enters `state_1`, and then stays there forever.

```python
states = ["state_1"] # a list of all of the states in the task
events = [] # a list of all of the events that the task will respond to

# the state that the task will start out in
initial_state = "state_1" 

# the state function that will be called every time there is an event
def state_1(event): 
    pass
```

## 2. Adding a device 

This task still doesn't do anything, but we've laid the groundwork for responding inputs and producing outputs using a device.

```python
from devices import Breakout_1_2, Poke # (1)

# create instances of devices
bb = Breakout_1_2()  # breakout board (2)
center_poke = Poke(bb.port_2, rising_event="center_in") # (3)

states = ["state_1"]
events = ["center_in"] # (4)

initial_state = "state_1"

def state_1(event):
    pass
```

1. [Device drivers](https://github.com/pyControl/code/tree/master/devices) make it easy to use hardware by abstracting away the lower level details of pin mappings and communication protocols, and providing a user-friendly interface to the hardware.

2. This [breakout board](https://pycontrol.readthedocs.io/en/latest/user-guide/hardware/#breakout-boards) class maps the microcontroller GPIO pins to the breakout board's connectors (RJ45 and BNC) and LEDs. This makes writing code easier as you can simply pass in a port nuumber into a device class to access the hardware.

3. We specify which port the device is plugged in to by passing in `bb.port_2` and give an event name `"center_in"` that will be emitted when the poke is detected.

4. Any events that we plan on using should be added to the `events` list.

## 3. Transition states

At any given time, the framework is in one state and all events will be processed through that state's function of the same name.

Here we define two states `state_1` and `state_2`. 
We will respond to events coming from the nosepoke devices to transition between states.

When the the state machine is in `state_1`, all it cares about is responding to a `center_in` event.
Other events, should they occur, will still be logged to the data log, but only the `center_in` will be acted upon by the state machine while in `state_1`.

When in `state_2`, all it cares about is responding to a `left_in` event.

```python

from pyControl.utility import *
from devices import Breakout_1_2, Poke

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in")

states = ["state_1", "state_2"]
events = ["center_in", "left_in"] 

initial_state = "state_1"


def state_1(event):
    if event == "center_in": # (1)
        goto_state("state_2") # (2)


def state_2(event):
    if event == "left_in":
        goto_state("state_1")
```

1. We check if the event is `"center_in"` and if so, we transition to `state_2`.
2. We use the `goto_state` function to transition to a new state.

## 4. Outputs

So far we have logged events and transitioned between states, but we still haven't altered the physical environment.
Here we make use of a [nosepoke](https://pycontrol.readthedocs.io/en/latest/user-guide/hardware/#poke) device's `LED` attribute to turn on and off the LED when the state machine is in `state_1` and `state_2` respectively.

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in")

states = ["state_1", "state_2"]
events = ["center_in", "left_in"]

initial_state = "state_1"


def state_1(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        goto_state("state_2")


def state_2(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        goto_state("state_1")
```

## 5. Variables

[Variables](https://pycontrol.readthedocs.io/en/latest/user-guide/programming-tasks/#variables) can be used througout the task.

Here we define two variables `poke_count___` and `poke_threshold`.
When in `state_1` we increment the `poke_count___` each time the center nosepoke is entered. After 4 pokes, we reset the poke count and transition to `state_2`.

All variables can be modified with the `controls` dialog.

For "private" variables that the experimenter doesn't need to adjust, it's best practice to add 3 underscores at the end of the variable name. This keeps the controls dialog uncluttered by hiding  unneccesary private variable controls.




```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in")

states = ["state_1", "state_2"]
events = ["center_in", "left_in"]

initial_state = "state_1"


# variables
v.poke_count___ = 0
v.poke_threshold = 4


def state_1(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count___ += 1
        if v.poke_count___ >= v.poke_threshold:
            v.poke_count___ = 0 # reset the poke count
            goto_state("state_2")


def state_2(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        goto_state("state_1")
```

## 6. Timers

Read more about time dependent behaviour [here](https://pycontrol.readthedocs.io/en/latest/user-guide/programming-tasks/#time-dependent-behaviour).

```python title="tasks/demo.py" hl_lines="23-24" linenums="1"
from pyControl.utility import *
from devices import Breakout_1_2, Poke

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")

states = ["state_1", "state_2"]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
]

initial_state = "state_1"


# variables
v.poke_count___ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number = 0


def state_1(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count___ += 1
        print("{} pokes".format(v.poke_count___))
        if v.poke_count___ >= v.poke_threshold:
            v.poke_count___ = 0
            goto_state("state_2")


def state_2(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")
        print("not held long enough")
    elif event == "left_hold_complete":
        v.trial_number += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}".format(
                v.trial_number, v.poke_count___, v.left_hold_duration
            )
        )
        goto_state("state_1")
```

## 7. Analog data

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke, Rotary_encoder

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")
running_wheel = Rotary_encoder(
    name="running_wheel",
    sampling_rate=40,
    output="velocity",
)

states = ["state_1", "state_2"]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
]

initial_state = "state_1"


# variables
v.poke_count___ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number = 0


def state_1(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count___ += 1

        if v.poke_count___ >= v.poke_threshold:
            v.poke_count___ = 0
            goto_state("state_2")


def state_2(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")

    elif event == "left_hold_complete":
        v.trial_number += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}".format(
                v.trial_number, v.poke_count___, v.left_hold_duration
            )
        )
        goto_state("state_1")
```

## 8. Analog Threshold

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke, Rotary_encoder

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")
running_wheel = Rotary_encoder(
    name="running_wheel",
    sampling_rate=40,
    output="velocity",
    threshold=1000,
    rising_event="started_running",
    falling_event="stopped_running",
)

states = [
    "want_pokes",
    "want_hold",
    "bonus_state",
]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
    "started_running",
    "stopped_running",
    "ran_enough",
]

initial_state = "want_pokes"


# variables
v.poke_count___ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number = 0
v.ran_enough_duration = 3


def want_pokes(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count___ += 1
        if v.poke_count___ >= v.poke_threshold:
            v.poke_count___ = 0
            goto_state("want_hold")


def want_hold(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")
    elif event == "left_hold_complete":
        goto_state("want_pokes")
    elif event == "started_running":
        set_timer("ran_enough", v.ran_enough_duration * second)
    elif event == "stopped_running":
        disarm_timer("ran_enough")
    elif event == "ran_enough":
        goto_state("bonus_state")
    elif event == "exit":
        v.trial_number += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}".format(
                v.trial_number, v.poke_count___, v.left_hold_duration
            )
        )


def bonus_state(event):
    if event == "entry":
        print("!!!!!!!!!!!!!ExTrA BoNus ReWArD!!!!!!!!!!")
        timed_goto_state("want_pokes", 2 * second)

```

## 9. Timed session

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke, Rotary_encoder

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")
running_wheel = Rotary_encoder(
    name="running_wheel",
    sampling_rate=40,
    output="velocity",
    threshold=10,
    rising_event="started_running",
    falling_event="stopped_running",
)

states = [
    "want_pokes",
    "want_hold",
    "bonus_state",
]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
    "started_running",
    "stopped_running",
    "ran_enough",
    "session_timer",
]

initial_state = "want_pokes"


# variables
v.poke_count___ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number = 0
v.ran_enough_duration = 3
v.session_duration = 10 


def run_start():
    print("called at the start of the session")
    set_timer("session_timer", v.session_duration * second)


def run_end():
    print("called at the end of the session")


def all_states(event):
    # When 'session_timer' event occurs stop framework to end session.
    if event == "session_timer":
        stop_framework()


def want_pokes(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count___ += 1
        if v.poke_count___ >= v.poke_threshold:
            v.poke_count___ = 0
            goto_state("want_hold")


def want_hold(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")
    elif event == "left_hold_complete":
        goto_state("want_pokes")
    elif event == "started_running":
        set_timer("ran_enough", v.ran_enough_duration * second)
    elif event == "stopped_running":
        disarm_timer("ran_enough")
    elif event == "ran_enough":
        goto_state("bonus_state")
    elif event == "exit":
        v.trial_number += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}".format(
                v.trial_number, v.poke_count___, v.left_hold_duration
            )
        )


def bonus_state(event):
    if event == "entry":
        print("!!!!!!!!!!!!!ExTrA BoNus ReWArD!!!!!!!!!!")
        timed_goto_state("want_pokes", 2 * second)

```

## 10. Hardware variables

Read more about hardware variables [here](https://pycontrol.readthedocs.io/en/latest/user-guide/programming-tasks/#special-variables) 

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke, Rotary_encoder

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")
running_wheel = Rotary_encoder(
    name="running_wheel",
    sampling_rate=40,
    output="velocity",
    threshold=10,
    rising_event="started_running",
    falling_event="stopped_running",
)

states = [
    "want_pokes",
    "want_hold",
    "bonus_state",
]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
    "started_running",
    "stopped_running",
    "ran_enough",
    "session_timer",
]

initial_state = "want_pokes"


# variables
v.poke_count___ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number = 0
v.ran_enough_duration = 3
v.session_duration = 10

v.hw_friction_factor = None


def run_start():
    print("called at the start of the session")
    set_timer("session_timer", v.session_duration * second)


def run_end():
    print("called at the end of the session")


def all_states(event):
    # When 'session_timer' event occurs stop framework to end session.
    if event == "session_timer":
        stop_framework()


def want_pokes(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count___ += 1
        if v.poke_count___ >= v.poke_threshold:
            v.poke_count___ = 0
            goto_state("want_hold")


def want_hold(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")
    elif event == "left_hold_complete":
        goto_state("want_pokes")
    elif event == "started_running":
        set_timer("ran_enough", v.ran_enough_duration / v.hw_friction_factor * second)
    elif event == "stopped_running":
        disarm_timer("ran_enough")
    elif event == "ran_enough":
        goto_state("bonus_state")
    elif event == "exit":
        v.trial_number += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}, Run:{}".format(
                v.trial_number,
                v.poke_count___,
                v.left_hold_duration,
                v.ran_enough_duration / v.hw_friction_factor,
            )
        )


def bonus_state(event):
    if event == "entry":
        print("!!!!!!!!!!!!!ExTrA BoNus ReWArD!!!!!!!!!!")
        timed_goto_state("want_pokes", 2 * second)
```

## 11. Custom controls

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke, Rotary_encoder

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")
running_wheel = Rotary_encoder(
    name="running_wheel",
    sampling_rate=40,
    output="velocity",
    threshold=10,
    rising_event="started_running",
    falling_event="stopped_running",
)

states = [
    "want_pokes",
    "want_hold",
    "bonus_state",
]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
    "started_running",
    "stopped_running",
    "ran_enough",
    "session_timer",
]

initial_state = "want_pokes"


# variables
v.poke_count______ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number___ = 0
v.ran_enough_duration = 3
v.session_duration = 10

v.hw_friction_factor = None
v.custom_controls_dialog = "andys_controls_UI"


def run_start():
    print("called at the start of the session")
    set_timer("session_timer", v.session_duration * second)


def run_end():
    print("called at the end of the session")


def all_states(event):
    # When 'session_timer' event occurs stop framework to end session.
    if event == "session_timer":
        stop_framework()


def want_pokes(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count______ += 1
        if v.poke_count______ >= v.poke_threshold:
            v.poke_count______ = 0
            goto_state("want_hold")


def want_hold(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")
    elif event == "left_hold_complete":
        goto_state("want_pokes")
    elif event == "started_running":
        set_timer("ran_enough", v.ran_enough_duration / v.hw_friction_factor * second)
    elif event == "stopped_running":
        disarm_timer("ran_enough")
    elif event == "ran_enough":
        goto_state("bonus_state")
    elif event == "exit":
        v.trial_number___ += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}, Run:{}".format(
                v.trial_number,
                v.poke_count___,
                v.left_hold_duration,
                v.ran_enough_duration / v.hw_friction_factor,
            )
        )


def bonus_state(event):
    if event == "entry":
        print("!!!!!!!!!!!!!ExTrA BoNus ReWArD!!!!!!!!!!")
        timed_goto_state("want_pokes", 2 * second)
```

## 12. API

```python title="tasks/demo.py"
from pyControl.utility import *
from devices import Breakout_1_2, Poke, Rotary_encoder

# create instances of devices
bb = Breakout_1_2()  # breakout board
center_poke = Poke(bb.port_2, rising_event="center_in")
left_poke = Poke(bb.port_3, rising_event="left_in", falling_event="left_out")
running_wheel = Rotary_encoder(
    name="running_wheel",
    sampling_rate=40,
    output="velocity",
    threshold=10,
    rising_event="started_running",
    falling_event="stopped_running",
)

states = [
    "want_pokes",
    "want_hold",
    "bonus_state",
]
events = [
    "center_in",
    "left_in",
    "left_out",
    "left_hold_complete",
    "started_running",
    "stopped_running",
    "ran_enough",
    "session_timer",
]

initial_state = "want_pokes"


# variables
v.poke_count______ = 0
v.poke_threshold = 5
v.left_hold_duration = 1
v.trial_number___ = 0
v.ran_enough_duration = 3
v.session_duration = 10

v.hw_friction_factor = None
v.custom_controls_dialog = "andys_controls_UI"
v.api_class = "Demo_api"


def run_start():
    print("called at the start of the session")
    set_timer("session_timer", v.session_duration * second)


def run_end():
    print("called at the end of the session")


def all_states(event):
    # When 'session_timer' event occurs stop framework to end session.
    if event == "session_timer":
        stop_framework()


def want_pokes(event):
    if event == "entry":
        center_poke.LED.on()
        left_poke.LED.off()
    elif event == "center_in":
        v.poke_count______ += 1
        if v.poke_count______ >= v.poke_threshold:
            v.poke_count______ = 0
            goto_state("want_hold")


def want_hold(event):
    if event == "entry":
        center_poke.LED.off()
        left_poke.LED.on()
    elif event == "left_in":
        set_timer("left_hold_complete", v.left_hold_duration * second)
    elif event == "left_out":
        disarm_timer("left_hold_complete")
    elif event == "left_hold_complete":
        goto_state("want_pokes")
    elif event == "started_running":
        set_timer("ran_enough", v.ran_enough_duration / v.hw_friction_factor * second)
    elif event == "stopped_running":
        disarm_timer("ran_enough")
    elif event == "ran_enough":
        goto_state("bonus_state")
    elif event == "exit":
        v.trial_number___ += 1
        print(
            "Trial:{}, Poke:{}, Hold:{}, Run:{}".format(
                v.trial_number,
                v.poke_count___,
                v.left_hold_duration,
                v.ran_enough_duration / v.hw_friction_factor,
            )
        )


def bonus_state(event):
    if event == "entry":
        print("!!!!!!!!!!!!!ExTrA BoNus ReWArD!!!!!!!!!!")
        timed_goto_state("want_pokes", 2 * second)
```

```python title="api_classes/Demo_api.py"
from source.gui.api import Api
from telegram import Bot
import asyncio
import threading
import matplotlib.pyplot as plt


TOKEN = "7527948084:AAHCVuQ7bKdvYIoEAb57dxyp0hqxa5dkt5o"
CHAT_ID = -4740996122

plt.rcParams["toolbar"] = "None"  # Disable Matplotlib figure toolbar.
plt.rc("axes.spines", top=False, right=False)  # Disable top and right axis spines.
plt.switch_backend("Qt6Agg")


class Telegram:
    def __init__(self, token, chatID):
        self.bot = Bot(token=token)
        self.chat_id = chatID
        self.loop = asyncio.new_event_loop()
        self.thread = threading.Thread(target=self._run_event_loop, daemon=True)
        self.thread.start()

    def _run_event_loop(self):
        asyncio.set_event_loop(self.loop)
        self.loop.run_forever()

    async def async_msg_send(self, msg):
        async with self.bot:
            await self.bot.send_message(
                text=msg, chat_id=self.chat_id, parse_mode="HTML"
            )

    def notify(self, *message_lines, wait_for_send=False):
        msg = "\n".join(message_lines)
        future = asyncio.run_coroutine_threadsafe(self.async_msg_send(msg), self.loop)
        if wait_for_send:
            future.result()

    def test(self):
        self.notify("This is a test notification from pyControl settings")

    def stop(self):
        self.loop.call_soon_threadsafe(self.loop.stop)
        self.thread.join()


# This class should be have the same name as the file and inherit the API class
# look at source/gui/api.py to see what functions can be redefined and called
class Demo_api(Api):
    def __init__(self):
        self.off_count = 0
        self.telegrammer = None

        self.figure = plt.figure()

    # this runs at the start of sessoin
    def run_start(self):
        self.ax = self.figure.add_subplot(111)
        self.pokes = ["left", "center"]
        self.pokes_count = [0, 0]
        self.ax.bar(self.pokes, self.pokes_count)
        plt.show(block=False)

        if hasattr(self.board.data_logger, "setup_ID"):
            self.telegrammer = Telegram(TOKEN, CHAT_ID)
        else:
            self.print_to_log("No setup ID found")

    # use this function
    def process_data_user(self, data):
        new_events = [new_event.name for new_event in data["events"]]
        for event in new_events:
            if event == "left_in":
                self.pokes_count[0] += 1
            elif event == "center_in":
                self.pokes_count[1] += 1

        # Clear the current bars and redraw with updated counts
        self.ax.clear()
        self.ax.set_title("Poke Counts")
        self.ax.set_ylim((0, max(self.pokes_count) + 1))
        self.ax.bar(self.pokes, self.pokes_count, color=["red", "blue"])
        self.figure.canvas.draw_idle()
        self.figure.canvas.flush_events()

    def run_stop(self):
        self.print_to_log("\nMessage from API at the end of the session")
        if self.telegrammer:
            setup, subject = (
                self.board.data_logger.setup_ID,
                self.board.data_logger.subject_ID,
            )
            final_vars = self.board.get_variables()
            session_duration = self.board.timestamp // 1000
            self.telegrammer.notify(
                "Session complete!",
                f"Subject {subject} in {setup}",
                f"{session_duration // 3600:02d}h {(session_duration % 3600) // 60:02d}m {session_duration % 60:02d}s",
                f"{final_vars['trial_number___']} trial{'s' if final_vars['trial_number___'] != 1 else ''} completed",
            )
```
