import machine
import utime
import random
from machine import Pin, I2C
from ssd1306 import SSD1306_I2C

# Initialize I2C for OLED display
i2c = I2C(1, scl=Pin(15), sda=Pin(14), freq=400000)
WIDTH = 128
HEIGHT = 64
oled = SSD1306_I2C(WIDTH, HEIGHT, i2c)

# Define buttons
up = Pin(2, Pin.IN, Pin.PULL_UP)
down = Pin(3, Pin.IN, Pin.PULL_UP)
left = Pin(4, Pin.IN, Pin.PULL_UP)
right = Pin(5, Pin.IN, Pin.PULL_UP)
shoot = Pin(6, Pin.IN, Pin.PULL_UP)

# Game parameters
crosshair_size = 3
crosshair_x = WIDTH // 2
crosshair_y = HEIGHT // 2
crosshair_speed = 3  # Pixels per update

target_size = 8
targets = []
target_spawn_interval = 1  # Number of seconds between target spawns
target_lifetime = 1.3  # Seconds each target stays on screen
last_spawn_time = utime.ticks_ms()

score = 0
game_time = 45  # Game duration in seconds
start_time = utime.ticks_ms()

# Draw the crosshair
def draw_crosshair(x, y):
    oled.fill_rect(x - crosshair_size, y - crosshair_size, crosshair_size * 2, crosshair_size * 2, 1)

# Draw a target
def draw_target(x, y):
    oled.fill_rect(x, y, target_size, target_size, 1)

# Draw the remaining time
def draw_timer(remaining_time):
    oled.text(f"Time: {remaining_time:2d}s", 0, 8)

# Update targets
def update_targets(current_time):
    global score
    new_targets = []
    for (x, y, spawn_time) in targets:
        if utime.ticks_diff(current_time, spawn_time) < target_lifetime * 1000:
            new_targets.append((x, y, spawn_time))
        else:
            score -= 1  # Decrease score if a target disappears
    return new_targets

# Check for collisions
def check_collisions():
    global targets, score
    new_targets = []
    for (x, y, spawn_time) in targets:
        if crosshair_x - crosshair_size <= x + target_size and crosshair_x + crosshair_size >= x and \
           crosshair_y - crosshair_size <= y + target_size and crosshair_y + crosshair_size >= y:
            score += 1
        else:
            new_targets.append((x, y, spawn_time))
    targets = new_targets

# Main game loop
def game_loop():
    global crosshair_x, crosshair_y, targets, start_time, last_spawn_time

    while True:
        current_time = utime.ticks_ms()
        elapsed_time = (utime.ticks_diff(current_time, start_time)) / 1000
        remaining_time = max(0, game_time - int(elapsed_time))

        if remaining_time <= 0:
            oled.fill(0)
            oled.text("Game Over!", 30, 20)
            oled.text(f"Score: {score}", 30, 30)
            oled.show()
            utime.sleep(5)
            break

        # Movement
        if not up.value():
            crosshair_y = max(crosshair_y - crosshair_speed, crosshair_size)
        if not down.value():
            crosshair_y = min(crosshair_y + crosshair_speed, HEIGHT - crosshair_size)
        if not left.value():
            crosshair_x = max(crosshair_x - crosshair_speed, crosshair_size)
        if not right.value():
            crosshair_x = min(crosshair_x + crosshair_speed, WIDTH - crosshair_size)

        # Spawn targets at intervals
        if utime.ticks_diff(current_time, last_spawn_time) > target_spawn_interval * 1000:
            x = random.randint(0, WIDTH - target_size)
            y = random.randint(0, HEIGHT - target_size)
            targets.append((x, y, current_time))
            last_spawn_time = current_time

        # Update and draw game state
        targets = update_targets(current_time)
        check_collisions()

        oled.fill(0)
        draw_crosshair(crosshair_x, crosshair_y)
        for (x, y, _) in targets:
            draw_target(x, y)
        draw_timer(remaining_time)
        oled.text("Score: {}".format(score), 0, 0)
        oled.show()
        utime.sleep(0.01)  # Short delay for smoother rendering

# Start the game
game_loop()
