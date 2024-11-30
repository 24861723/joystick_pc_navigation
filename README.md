# Joystick PC Navigation

This Python script allows you to control your mouse using a joystick. It utilizes Pygame for joystick input and PyAutoGUI for mouse actions.

## Features
- Click and scroll using joystick buttons.
- Zoom in/out with specific button presses.
- Smooth mouse movement based on joystick axis.

## Requirements
- Python 3.x
- Pygame
- PyAutoGUI

## Installation
```bash
pip install pygame pyautogui
```

## Usage
1. Connect your joystick.
2. Run the script:
   ```bash
   python joystick_control.py
   ```
3. Use the joystick to move the mouse and buttons to perform actions.

## Notes
- Adjust sensitivity and dead zone in the code as needed.
- Ensure proper permissions for mouse control on your OS.

---
## Quick Look
```python
import pygame
import pyautogui

# Initialize pygame
pygame.init()
pygame.joystick.init()

# Check if a joystick is connected
if pygame.joystick.get_count() == 0:
    print("No joystick connected")
    exit()

# Initialize the first joystick
joystick = pygame.joystick.Joystick(0)
joystick.init()

# Calibration variables for sensitivity and smoothness
sensitivity = 8000     # High sensitivity as per your choice
scroll_sensitivity = 12000
smoothness = 180       # High FPS for smoother movement
dead_zone = 0.1        # Dead zone threshold

# Set up the clock for controlling the loop speed
clock = pygame.time.Clock()
fps = smoothness       # Frames per second

# Initialize cumulative movement variables
cum_move_x = 0.0
cum_move_y = 0.0
cum_scroll = 0.0

# Initialize previous button states for debouncing
prev_button_states = [False] * joystick.get_numbuttons()

# Main loop
try:
    while True:
        # Process events (improves performance by handling events properly)
        for event in pygame.event.get():
            pass  # We can handle specific events here if needed

        # Handle button presses with debouncing
        buttons = joystick.get_numbuttons()
        for i in range(buttons):
            current_state = joystick.get_button(i)
            prev_state = prev_button_states[i]
            if not prev_state and current_state:
                # Button i was just pressed
                if i == 5:
                    pyautogui.click(button='left')
                elif i == 7:
                    pyautogui.click(button='right')
                elif i == 0:
                    pyautogui.hotkey('ctrl', '+')  # Zoom in
                elif i == 2:
                    pyautogui.hotkey('ctrl', '-')  # Zoom out
                elif i == 3:
                    pyautogui.press('left')        # Left arrow key
                elif i == 1:
                    pyautogui.press('right')       # Right arrow key
                elif i == 4:
                    pyautogui.hotkey('win', 'tab') # Windows + Tab
                elif i == 6:
                    pyautogui.press('f11')         # Toggle Fullscreen

            # Update previous state
            prev_button_states[i] = current_state

        # Handle stick movements
        axis_x = joystick.get_axis(0)
        axis_y = joystick.get_axis(1)
        axis_scroll = joystick.get_axis(2)

        # Apply dead zone to prevent drift
        if abs(axis_x) < dead_zone:
            axis_x = 0.0
        if abs(axis_y) < dead_zone:
            axis_y = 0.0
        if abs(axis_scroll) < dead_zone:
            axis_scroll = 0.0

        # Normalize movement based on frame rate
        move_x = axis_x * sensitivity / fps
        move_y = axis_y * sensitivity / fps
        scroll_amount = axis_scroll * scroll_sensitivity / fps

        # Accumulate movement
        cum_move_x += move_x
        cum_move_y += move_y
        cum_scroll += scroll_amount

        # Extract integer part for actual movement
        int_move_x = int(cum_move_x)
        int_move_y = int(cum_move_y)
        int_scroll = int(cum_scroll)

        # Update cumulative movement by removing the integer part
        cum_move_x -= int_move_x
        cum_move_y -= int_move_y
        cum_scroll -= int_scroll

        # Move the mouse if there is any movement
        if int_move_x != 0 or int_move_y != 0:
            pyautogui.moveRel(int_move_x, int_move_y)

        # Scroll the mouse if there is any scroll input
        if int_scroll != 0:
            pyautogui.scroll(-int_scroll)

        # Control the loop speed
        clock.tick(fps)

except KeyboardInterrupt:
    pass

# Quit pygame
pygame.quit()
