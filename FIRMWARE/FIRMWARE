# I2C
SDA = 1
SCL = 2

# Buttons
SW1 = 3
SW2 = 4
SW3 = 2

# Rotary encoder
ENC_A  = 6
ENC_B  = 7
ENC_SW = 1


from machine import Pin, I2C
import time

# ===================== I2C + PCA9555 =====================

class PCA9555:
    def __init__(self, i2c, addr):
        self.i2c = i2c
        self.addr = addr
        self.i2c.writeto_mem(addr, 0x06, b'\x00\x00')  # all outputs

    def write(self, p0, p1):
        self.i2c.writeto_mem(self.addr, 0x02, bytes([p0, p1]))

# ===================== SETUP =====================

i2c = I2C(0, sda=Pin(SDA), scl=Pin(SCL), freq=400000)

pca_4dig = PCA9555(i2c, 0x20)
pca_1dig = PCA9555(i2c, 0x21)

sw1 = Pin(SW1, Pin.IN, Pin.PULL_UP)
sw2 = Pin(SW2, Pin.IN, Pin.PULL_UP)
sw3 = Pin(SW3, Pin.IN, Pin.PULL_UP)

enc_a  = Pin(ENC_A,  Pin.IN, Pin.PULL_UP)
enc_b  = Pin(ENC_B,  Pin.IN, Pin.PULL_UP)
enc_sw = Pin(ENC_SW, Pin.IN, Pin.PULL_UP)

# ===================== SEGMENTS (COMMON ANODE) =====================

SEG = {
    0: 0b11000000,
    1: 0b11111001,
    2: 0b10100100,
    3: 0b10110000,
    4: 0b10011001,
    5: 0b10010010,
    6: 0b10000010,
    7: 0b11111000,
    8: 0b10000000,
    9: 0b10010000
}

# ===================== STATE =====================

score_u5 = 0
score_u7 = 0

minutes = 0
seconds = 0

timer_running = False
flashing = False

last_enc = enc_a.value()
last_tick = time.ticks_ms()

# ===================== DISPLAY =====================

def display_4digit(digits):
    for i, d in enumerate(digits):
        pca_4dig.write(SEG[d], ~(1 << i) & 0xFF)
        time.sleep_ms(2)

def display_scores():
    s1 = score_u5 % 10
    s2 = score_u7 % 10
    pca_1dig.write(SEG[s1], 0b00000001)
    pca_1dig.write(SEG[s2], 0b00000010)

def display_time():
    m1 = minutes // 10
    m2 = minutes % 10
    s1 = seconds // 10
    s2 = seconds % 10
    display_4digit([m1, m2, s1, s2])

def clear_display():
    pca_4dig.write(0xFF, 0xFF)
    pca_1dig.write(0xFF, 0xFF)

# ===================== INPUT HANDLERS =====================

def handle_buttons():
    global score_u5, score_u7, minutes, seconds, timer_running

    if flashing:
        return

    if not sw1.value():
        score_u5 += 1
        time.sleep(0.2)

    if not sw2.value():
        score_u7 += 1
        time.sleep(0.2)

    if not sw3.value():
        score_u5 = 0
        score_u7 = 0
        minutes = 0
        seconds = 0
        timer_running = False
        time.sleep(0.3)

    if not enc_sw.value():
        timer_running = True
        time.sleep(0.3)

def handle_encoder():
    global minutes, last_enc
    curr = enc_a.value()
    if curr != last_enc:
        if enc_b.value() != curr:
            minutes = (minutes + 1) % 61
        else:
            minutes = (minutes - 1) % 61
        last_enc = curr

# ===================== TIMER =====================

def update_timer():
    global minutes, seconds, timer_running, flashing, last_tick

    if not timer_running:
        return

    now = time.ticks_ms()
    if time.ticks_diff(now, last_tick) >= 1000:
        last_tick = now

        if minutes == 0 and seconds == 0:
            timer_running = False
            flashing = True
            flash_zero()
            return

        if seconds == 0:
            minutes -= 1
            seconds = 59
        else:
            seconds -= 1

def flash_zero():
    global flashing
    for _ in range(10):
        display_4digit([0,0,0,0])
        time.sleep(0.25)
        clear_display()
        time.sleep(0.25)
    flashing = False

# ===================== MAIN LOOP =====================

while True:
    handle_buttons()
    handle_encoder()
    update_timer()

    display_time()
    display_scores()
