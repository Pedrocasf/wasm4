# Handling User Input

import MultiLanguageCode from '@site/src/components/MultiLanguageCode';

## Gamepad

The main input device is the gamepad, consisting of 4 directions and 2 action
buttons. On computers, players use the arrow keys and the X and Z keys. On
mobile, a virtual gamepad overlay is displayed that players can tap.

The gamepad state is stored by WASM-4 as one byte in memory, with each button
stored as a single bit. For example, the right directional button is stored in
bit 5. We can mask the gamepad with the `BUTTON_RIGHT` constant to check if the
right button is pressed.

<MultiLanguageCode>

```typescript
const gamepad = load<u8>(w4.GAMEPAD1);

if (gamepad & w4.BUTTON_RIGHT) {
    w4.trace("Right button is down!");
}
```

```c
uint8_t gamepad = *GAMEPAD1;

if (gamepad & BUTTON_RIGHT) {
    trace("Right button is down!");
}
```

```c3
char gamepad = *w4::GAMEPAD1;

if (gamepad & w4::BUTTON_RIGHT) 
{
    w4::trace("Right button is down!");
}
```

```d
ubyte gamepad = *w4.gamepad1;

if (gamepad & w4.buttonRight) {
    w4.trace("Right button is down!");
}
```

```go
var gamepad = *w4.GAMEPAD1

if gamepad&w4.BUTTON_RIGHT != 0 {
    w4.Trace("Right button is down!")
}
```

```lua
local gamepad = $GAMEPAD1

if gamepad & BUTTON_RIGHT ~= 0 then
    trace("Right button is down!")
end
```

```nim
var gamepad = GAMEPAD1[]

if bool(gamepad and BUTTON_RIGHT):
  trace("Right button is down!")
```

```odin
if .RIGHT in w4.GAMEPAD1^ {
    w4.trace("Right button is down!")
}
```

```penne
if GAMEPAD1 & BUTTON_RIGHT != 0
{
    trace("Right button is down!");
}
```

```porth
memory gamepad sizeof(u8) end
$GAMEPAD1 @8 gamepad !8

$gamepad @8 $BUTTON_RIGHT and cast(bool) if
  "Right button is down!"c trace
end
```

```roland
if GAMEPADS~[0] & BUTTON_RIGHT != 0 {
   trace("Right button is down!");
}
```

```rust
let gamepad = unsafe { *GAMEPAD1 };

if gamepad & BUTTON_RIGHT != 0 {
    trace("Right button is down!");
}
```

```wasm
(data (i32.const 0x2000) "Right button is down!\00")

(local $gamepad i32)
(local.set $gamepad (i32.load8_u (global.get $GAMEPAD1)))

(if (i32.and (local.get $gamepad) (global.get $BUTTON_RIGHT))
  (then
    (call $trace (i32.const 0x2000))
  )
)
```

```zig
const gamepad = w4.GAMEPAD1.*;

if (gamepad & w4.BUTTON_RIGHT != 0) {
    w4.trace("Right button is down!");
}
```

</MultiLanguageCode>

| Gamepad Bit | Button                |
| ---         | ---                   |
| 0           | `BUTTON_1` (1)        |
| 1           | `BUTTON_2` (2)        |
| 2           | *Unused*              |
| 3           | *Unused*              |
| 4           | `BUTTON_LEFT` (16)    |
| 5           | `BUTTON_RIGHT` (32)   |
| 6           | `BUTTON_UP` (64)      |
| 7           | `BUTTON_DOWN` (128) |

### Checking if a button was pressed this frame

`GAMEPAD1` stores the current state of the gamepad. It's common to want to know
if a button was just pressed this frame. Another way of putting it: if a
button was *not* down last frame but *is* down this frame.

This can be handled by storing the previous gamepad state, and then [bitwise
XORing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_XOR) (`^`)
it with the current gamepad state. This leaves us with a byte with only the
buttons that were pressed this frame.

<MultiLanguageCode>

```typescript
let previousGamepad: u8;

export function update () {
    const gamepad = load<u8>(w4.GAMEPAD1);

    // Only the buttons that were pressed down this frame
    const pressedThisFrame = gamepad & (gamepad ^ previousGamepad);
    previousGamepad = gamepad;

    if (pressedThisFrame & w4.BUTTON_RIGHT) {
        trace("Right button was just pressed!");
    }
}
```

```c
uint8_t previousGamepad;

void update () {
    uint8_t gamepad = *GAMEPAD1;

    // Only the buttons that were pressed down this frame
    uint8_t pressedThisFrame = gamepad & (gamepad ^ previousGamepad);
    previousGamepad = gamepad;

    if (pressedThisFrame & BUTTON_RIGHT) {
        trace("Right button was just pressed!");
    }
}
```

```c3
char previous_gamepad;

fn void update () 
{
    char gamepad = *w4::GAMEPAD1;

    // Only the buttons that were pressed down this frame
    char pressed_this_frame = gamepad & (gamepad ^ previous_gamepad);
    previous_gamepad = gamepad;

    if (pressed_this_frame & w4::BUTTON_RIGHT) 
    {
        w4::trace("Right button was just pressed!");
    }
}
```

```d
ubyte previousGamepad;

void update() {
    ubyte gamepad = *w4.gamepad1;

    // Only the buttons that were pressed down this frame
    ubyte pressedThisFrame = gamepad & (gamepad ^ previousGamepad);
    previousGamepad = gamepad;

    if (pressedThisFrame & w4.buttonRight) {
        w4.trace("Right button was just pressed!");
    }
}
```

```go
var previousGamepad byte

//go:export update
func update () {
    var gamepad = *w4.GAMEPAD1

    // Only the buttons that were pressed down this frame
    var pressedThisFrame = gamepad & (gamepad ^ previousGamepad)
    previousGamepad = gamepad

    if pressedThisFrame&w4.BUTTON_RIGHT != 0 {
        w4.Trace("Right button was just pressed!")
    }
}
```

```lua
local previous_gamepad: uint8

local function update()
    local gamepad = $GAMEPAD1

    -- Only the buttons that were pressed down this frame
    local pressed_this_frame = gamepad & (gamepad ~ previous_gamepad)
    previous_gamepad = gamepad

    if pressed_this_frame & BUTTON_RIGHT ~= 0 then
        trace("Right button was just pressed!")
    end
end

## setup_wasm4_callbacks(update)
```

```nim
var previousGamepad: uint8

proc update {.exportWasm.} =
  var gamepad = GAMEPAD1[]

  # Only the buttons that were pressed down this frame
  var pressedThisFrame = gamepad and (gamepad xor previousGamepad);
  previousGamepad = gamepad

  if bool(pressedThisFrame and BUTTON_RIGHT):
    trace("Right button was just pressed!")
```

```odin
previous_gamepad : w4.Buttons

@export
update :: proc "c" () {
    gamepad := w4.GAMEPAD1^

    // Only the buttons that were pressed down this frame
    pressed_this_frame := gamepad & (gamepad ~ previous_gamepad)
    previous_gamepad = gamepad

    if .RIGHT in pressed_this_frame {
        w4.trace("Right button was just pressed!")
    }
```

```penne
const PREVIOUS_GAMEPAD: &u8 = 0xF700;

pub extern fn update()
{
    // Only the buttons that were pressed down this frame.
    var pressed_this_frame = GAMEPAD & (GAMEPAD ^ PREVIOUS_GAMEPAD);
    PREVIOUS_GAMEPAD = GAMEPAD;

    if pressed_this_frame & BUTTON_RIGHT != 0
    {
        trace("Right button was just pressed!");
    }
}
```

```porth
memory prev-state sizeof(u8) end

export update "update"
proc update in
  $GAMEPAD1 @8
  let gamepad in
    gamepad prev-state @8 xor gamepad and
    gamepad prev-state !8
    let pressed-this-frame in
      pressed-this-frame $BUTTON_RIGHT and cast(bool) if
        "Right button was just pressed!"c trace
      end
    end
  end
end
```

```roland
proc update() {
   let previous = PREVIOUS_GAMEPAD;
   let gamepad = GAMEPADS~[0];
   // Only the buttons that were pressed down this frame
   let pressed_this_frame = gamepad & (gamepad ^ previous);
   PREVIOUS_GAMEPAD = gamepad;

   if pressed_this_frame & BUTTON_RIGHT != 0 {
      trace("Right button was just pressed!");
   }
}
```

```rust
static mut PREVIOUS_GAMEPAD: u8 = 0;

#[no_mangle]
fn update() {
    let (pressed_this_frame, ..) = unsafe {
        let previous = PREVIOUS_GAMEPAD;
        let gamepad = *GAMEPAD1;
        // Only the buttons that were pressed down this frame
        let pressed_this_frame = gamepad & (gamepad ^ previous);
        PREVIOUS_GAMEPAD = gamepad;

        (pressed_this_frame, gamepad, previous)
    };

    if pressed_this_frame & BUTTON_RIGHT != 0 {
        trace("Right button was just pressed!");
    }
}
```

```wasm
;; Store the previous gamepad state in a global variable rather than linear
;; memory. It is defined as a mutable global with the initial value 0.
(global $previous-gamepad (mut i32) (i32.const 0))

(data (i32.const 0x2000) "Right button was just pressed!\00")

(func (export "update")
  (local $gamepad i32)
  (local $pressed-this-frame i32)

  (local.set $gamepad (i32.load8_u (global.get $GAMEPAD1)))

  ;; Only the buttons that were pressed down this frame
  (local.set $pressed-this-frame
    (i32.and
      (local.get $gamepad)
      (i32.xor
        (local.get $gamepad)
        (global.get $previous-gamepad))))

  (global.set $previous-gamepad (local.get $gamepad))

  (if (i32.and (local.get $pressed-this-frame) (global.get $BUTTON_RIGHT))
    (then
      (call $trace (i32.const 0x2000))
    )
  )
)
```

```zig
var previous_gamepad: u8 = 0;

export fn update() void {
    const gamepad = w4.GAMEPAD1.*;

    const pressed_this_frame = gamepad & (gamepad ^ previous_gamepad);
    previous_gamepad = gamepad;

    if (pressed_this_frame & w4.BUTTON_RIGHT != 0) {
        w4.trace("Right button was just pressed!");
    }
}
```

</MultiLanguageCode>

## Mouse

Mouse (or touchscreen) input is supported and will work for positions even outside of the game window on supported platforms.
See the [Memory Map](/docs/reference/memory) reference for more details on `MOUSE_X`, `MOUSE_Y`, and `MOUSE_BUTTONS`.

On the example below, we can make a rectangle follow the mouse position and expand when clicked:

<MultiLanguageCode>

```typescript
export function update (): void {
    const mouse  = load<u8>(w4.MOUSE_BUTTONS);
    const mouseX = load<i16>(w4.MOUSE_X);
    const mouseY = load<i16>(w4.MOUSE_Y);

    if (mouse & w4.MOUSE_LEFT) {
        store<u16>(w4.DRAW_COLORS, 4);
        w4.rect(mouseX - 8, mouseY - 8, 16, 16);
    } else {
        store<u16>(w4.DRAW_COLORS, 2);
        w4.rect(mouseX - 4, mouseY - 4, 8, 8);
    }
}
```

```c3
fn void update() @wasm()
{
    if (*w4::MOUSE & *w4::MOUSE_LEFT != 0)
    {
        *w4::DRAW_COLORS = 4;
        w4::rect(*w4::MOUSE_X - 8, *w4::MOUSE_Y - 8, 16, 16);
    }
    else
    {
        *w4::DRAW_COLORS = 2;
        w4::rect(*w4::MOUSE_X - 4, *w4::MOUSE_Y - 4, 8, 8);
    }
}
```

```penne
pub extern update()
{
    if MOUSE & MOUSE_LEFT != 0
    {
        DRAW_COLORS = 4;
        rect(MOUSE_X as i32 - 8, MOUSE_Y as i32 - 8, 16, 16);
    }
    else
    {
        DRAW_COLORS = 2;
        rect(MOUSE_X as i32 - 4, MOUSE_Y as i32 - 4, 8, 8);
    }
}
```

```rust
#[no_mangle]
fn update() {
    let mouse = unsafe { *MOUSE_BUTTONS };
    let mouse_x = unsafe { *MOUSE_X };
    let mouse_y = unsafe { *MOUSE_Y };

    if mouse & MOUSE_LEFT != 0 {
        unsafe { *DRAW_COLORS = 4 }
        rect(i32::from(mouse_x) - 8, i32::from(mouse_y) - 8, 16, 16);
    } else {
        unsafe { *DRAW_COLORS = 2 }
        rect(i32::from(mouse_x) - 4, i32::from(mouse_y) - 4, 8, 8);
    }
}
```

```zig
export fn update() void {
    const mouse = w4.MOUSE_BUTTONS.*;
    const mouseX = w4.MOUSE_X.*;
    const mouseY = w4.MOUSE_Y.*;

    if (mouse & w4.MOUSE_LEFT) {
        w4.DRAW_COLORS.* = 4;
        w4.rect(@as(i32, mouseX) - 8, @as(i32, mouseY) - 8, 16, 16);
    } else {
        w4.DRAW_COLORS.* = 2;
        w4.rect(@as(i32, mouseX) - 4, @as(i32, mouseY) - 4, 8, 8);
    }
}
```

</MultiLanguageCode>
