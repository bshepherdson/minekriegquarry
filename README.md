# Minekrieg Quarry Engine

Software to drive a quarry platform.

Code for the master computer lives in the `controller` directory, and code for the wireless mining turtles lives in the `turtle` directory.

## Overview

One complete sequence of operations runs as follows:
- Redstone signal to move the platform forward.
    - Wait (two seconds?) for this to be completed.
    - This wait time lives in `quarry/config/movetime`
- Signal the turtles to begin their digging.
    - Wireless signal on channel 666.
    - Turtles do the following:
        - Place their diggers (from slot 1)
        - Repeatedly clear their inventories. The period for clearing the inventory is given as a parameter in the message to start digging.
        - Wait for the signal from the controller to stop.
- Wait a certain length of time (5 seconds? 8? configured from `quarry/config/digtime`.
- Signal the turtles to retract their diggers.
    - Turtles clear their inventories one last time.
    - Turtles mine ahead of themselves, collecting the digger into slot 1.
    - TODO: Should the turtles signal back, or should it just wait?
- Wait a moment (2 seconds? `quarry/config/recoverytime`) for the turtles to collect their bits.
- Repeat from the beginning.


## Controller software

Uses configurable timing and repetitions, taking these values from files in `quarry/config/` on the controller computer.

- `movetime`: Seconds to wait for the frame to advance one block.
- `digtime`: Time to wait for the drills to proceed downward.
- `inventoryperiod`: Time between each clearing of the turtles' inventory.
- `recoverytime`: Seconds to wait for the turtles to empty their inventory one last time and then retrieve their drills.
- `repeat`: Number of cycles to complete before stopping. `0` for infinite.

## Messages to turtles

### Start digging

    { type: 'dig', period: number }

Start digging, and clear inventory every `period` seconds.

### Stop and recover drills

    { type: 'recovery' }


### Updated software delivery

    { type: 'update', url: string }

Sends a new copy of the software to be installed. The software should write the `content` to the file `/newversion` and then `os.reboot()`.

The `startup` program already installed on the turtles does the following:

    - Checks if `/newversion` exists.
        - If so, deletes `/quarry` and moves `/newversion` to `/quarry`.
        - Launches `/quarry`.


## Scripts on the Controller

### `quarrycontrol`

The main control program for the quarry. Run this to begin running the mining cycle.

### `update`

Sends the updated copy of `turtle` to the turtles.

### `fetch`

Downloads the latest versions of all the (other) software, excluding `fetch` itself.

