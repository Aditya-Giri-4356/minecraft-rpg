# Minecraft RPG

A fun 3D Minecraft-style RPG/RNG game built entirely in Python using Tkinter! 

### Controls:
- **W, A, S, D**: Move around
- **Right-Click & Drag**: Steer the camera
- **Q / E**: Turn the camera left or right
- **Space**: Jump (Double-tap to toggle Flying mode)
- **Left-Click**: Build blocks
- **F**: Till grass into farmland
- **G**: Plant seeds on farmland or harvest grown wheat
- **1-9, 0, P**: Select blocks from the dock

### 🌐 Play in your Browser (Cloud VM)
*Note: GitHub's security policies do not allow running VMs or interactive games directly inside the README page itself.*

However, you can play this game directly in your browser using a free Cloud VM on Replit! 
Click the button below to instantly spin up a VM and play the game online:

[![Run on Replit](https://replit.com/badge/github/Aditya-Giri-4356/minecraft-rpg)](https://replit.com/github/Aditya-Giri-4356/minecraft-rpg)

---

### Play it in IDLE!
You don't need to download any files if you don't want to. Because this uses standard Python (Tkinter), you can play it directly in Python IDLE!
Just copy the entire code block below, paste it into a new file in Python IDLE, and press Run (F5).

```python
import tkinter as tk
import math
import sys
import random
import time

# --- 1. Engine Configuration ---
SCALE = 35
GRID_SIZE = 40

target_angle = 225.0
current_angle = 225.0
pan_x = 500
pan_y = 325

MOVE_SPEED = 0.14
TURN_SPEED_KEY = 4.0
JUMP_VZ = 0.22
GRAVITY = 0.02
FLY_SPEED = 0.12
DOUBLE_TAP_WINDOW = 0.35

# Base Material Colors
COLORS = {
    'grass_top': (110, 205, 75),
    'grass_side': (95, 180, 60),
    'dirt': (150, 100, 55),
    'stone': (140, 140, 140),
    'cloud_top': (255, 255, 255),
    'cloud_side': (240, 240, 245),
    'cloud_bot': (220, 220, 225),
    'wood_top': (205, 170, 125),
    'wood_side': (139, 90, 43),
    'leaves': (34, 139, 34),
    'brick': (178, 34, 34),
    'sand': (238, 214, 175),
    'water': (65, 105, 225),
    'obsidian': (30, 15, 45),
    'farmland': (92, 64, 40),
    'wheat_0': (150, 190, 70),
    'wheat_1': (190, 200, 60),
    'wheat_2': (225, 195, 40),
    'plank': (193, 154, 107),
    # Steve materials
    'skin': (222, 172, 138),
    'shirt': (0, 158, 158),
    'pants': (60, 60, 150),
    'hair': (60, 40, 25),
    'eye_white': (255, 255, 255),
    'eye_pupil': (40, 40, 120),
}

# --- 2. Procedural Vector Models ---
def make_cube(c_top, c_side, c_bottom):
    return [
        (0, 1, 0, [(0,1,1), (1,1,1), (1,1,0), (0,1,0)], c_top, 'top'),
        (0, -1, 0, [(0,0,0), (1,0,0), (1,0,1), (0,0,1)], c_bottom, 'bottom'),
        (1, 0, 0, [(1,0,1), (1,1,1), (1,1,0), (1,0,0)], c_side, '+X'),
        (-1, 0, 0, [(0,0,0), (0,1,0), (0,1,1), (0,0,1)], c_side, '-X'),
        (0, 0, 1, [(0,0,1), (0,1,1), (1,1,1), (1,0,1)], c_side, '+Z'),
        (0, 0, -1, [(1,0,0), (1,1,0), (0,1,0), (0,0,0)], c_side, '-Z'),
    ]

def make_fluid(h):
    return [
        (0, 1, 0, [(0,h,1), (1,h,1), (1,h,0), (0,h,0)], 'water', 'top'),
        (0, -1, 0, [(0,0,0), (1,0,0), (1,0,1), (0,0,1)], 'water', 'bottom'),
        (1, 0, 0, [(1,0,1), (1,h,1), (1,h,0), (1,0,0)], 'water', '+X'),
        (-1, 0, 0, [(0,0,0), (0,h,0), (0,h,1), (0,0,1)], 'water', '-X'),
        (0, 0, 1, [(0,0,1), (0,h,1), (1,h,1), (1,0,1)], 'water', '+Z'),
        (0, 0, -1, [(1,0,0), (1,h,0), (0,h,0), (0,0,0)], 'water', '-Z'),
    ]

def make_slab(h, c_top, c_side, c_bottom):
    return [
        (0, 1, 0, [(0,h,1), (1,h,1), (1,h,0), (0,h,0)], c_top, 'top'),
        (0, -1, 0, [(0,0,0), (1,0,0), (1,0,1), (0,0,1)], c_bottom, 'bottom'),
        (1, 0, 0, [(1,0,1), (1,h,1), (1,h,0), (1,0,0)], c_side, '+X'),
        (-1, 0, 0, [(0,0,0), (0,h,0), (0,h,1), (0,0,1)], c_side, '-X'),
        (0, 0, 1, [(0,0,1), (0,h,1), (1,h,1), (1,0,1)], c_side, '+Z'),
        (0, 0, -1, [(1,0,0), (1,h,0), (0,h,0), (0,0,0)], c_side, '-Z'),
    ]

GRASS_MODEL = [
    (0, 1, 0, [(0,1,1), (1,1,1), (1,1,0), (0,1,0)], 'grass_top', 'top'),
    (0, -1, 0, [(0,0,0), (1,0,0), (1,0,1), (0,0,1)], 'dirt', 'bottom'),
    (1, 0, 0, [(1,0,1), (1,0.75,1), (1,0.75,0), (1,0,0)], 'dirt', '+X'),
    (1, 0, 0, [(1,0.75,1), (1,1,1), (1,1,0), (1,0.75,0)], 'grass_side', '+X'),
    (-1, 0, 0, [(0,0,0), (0,0.75,0), (0,0.75,1), (0,0,1)], 'dirt', '-X'),
    (-1, 0, 0, [(0,0.75,0), (0,1,0), (0,1,1), (0,0.75,1)], 'grass_side', '-X'),
    (0, 0, 1, [(0,0,1), (0,0.75,1), (1,0.75,1), (1,0,1)], 'dirt', '+Z'),
    (0, 0, 1, [(0,0.75,1), (0,1,1), (1,1,1), (1,0.75,1)], 'grass_side', '+Z'),
    (0, 0, -1, [(1,0,0), (1,0.75,0), (0,0.75,0), (0,0,0)], 'dirt', '-Z'),
    (0, 0, -1, [(1,0.75,0), (1,1,0), (0,1,0), (0,0.75,0)], 'grass_side', '-Z'),
]

BLOCK_MODELS = {
    '1': GRASS_MODEL,
    '2': make_cube('stone', 'stone', 'stone'),
    '3': make_cube('dirt', 'dirt', 'dirt'),
    '4': make_cube('cloud_top', 'cloud_side', 'cloud_bot'),
    '5': make_cube('wood_top', 'wood_side', 'wood_side'),
    '6': make_cube('leaves', 'leaves', 'leaves'),
    '7': make_cube('brick', 'brick', 'brick'),
    '8': make_cube('sand', 'sand', 'sand'),
    '0': make_cube('obsidian', 'obsidian', 'obsidian'),
    'P': make_cube('plank', 'plank', 'plank'),
    '9':   make_fluid(0.9),
    '9_S': make_fluid(0.9),
    '9_4': make_fluid(0.9),
    '9_3': make_fluid(0.65),
    '9_2': make_fluid(0.4),
    '9_1': make_fluid(0.15),
    'F':   make_slab(0.9, 'farmland', 'dirt', 'dirt'),
    'W0':  make_slab(0.35, 'wheat_0', 'wheat_0', 'farmland'),
    'W1':  make_slab(0.6, 'wheat_1', 'wheat_1', 'farmland'),
    'W2':  make_slab(0.85, 'wheat_2', 'wheat_2', 'farmland'),
}

BLOCK_NAMES = {
    '1': 'Grass', '2': 'Stone', '3': 'Dirt', '4': 'Snow', '5': 'Wood',
    '6': 'Leaves', '7': 'Brick', '8': 'Sand', '9': 'Water', '0': 'Obsidian',
    'P': 'Plank',
}

ADJACENT = {
    'top': (0, 1, 0), 'bottom': (0, -1, 0),
    '+X': (1, 0, 0), '-X': (-1, 0, 0),
    '+Z': (0, 0, 1), '-Z': (0, 0, -1)
}

NEIGHBOR_OFFSETS = [(1, 0, 0), (-1, 0, 0), (0, 0, 1), (0, 0, -1), (0, 1, 0), (0, -1, 0)]

current_block = '1'
world = {}
clouds_world = set()
SUN_DIR = (-0.4, 0.8, 0.5)

# --- Player state (Steve) ---
player_x = GRID_SIZE / 2
player_z = GRID_SIZE / 2
player_y = 1.0
player_vy = 0.0
player_facing = 225.0
flying = False
last_space_time = None
keys_held = set()
mouse_x = 500
mouse_y = 325

steve_swing_timer = 0  
steve_walk_cycle = 0.0

quest_state = {
    "till_done": False, "plant_done": False,
    "harvest_done": False, "house_done": False,
}
QUEST_STEPS = [
    ("till_done", "Step 1: Till a Grass block into Farmland (stand on grass, press F)"),
    ("plant_done", "Step 2: Plant Wheat on your Farmland (stand on farmland, press G)"),
    ("harvest_done", "Step 3: Wait for Wheat to fully grow, then harvest it (press G on ripe wheat)"),
    ("house_done", "Step 4: Build a house - place at least 4 Plank walls and 1 roof block"),
]
harvested_wheat = 0

# --- 3. Window Setup ---
root = tk.Tk()
root.title("Tkinter True 3D - Steve, Farming & Quests Edition")
root.geometry("1000x650")
canvas = tk.Canvas(root, bg="#87CEEB", highlightthickness=0)
canvas.pack(fill="both", expand=True)

color_cache = {}

def hex_color(rgb, intensity):
    r = int(max(0, min(255, rgb[0] * intensity)))
    g = int(max(0, min(255, rgb[1] * intensity)))
    b = int(max(0, min(255, rgb[2] * intensity)))
    return f"#{r:02x}{g:02x}{b:02x}"

def make_steve_parts():
    return [
        (-0.18, 0.0, 0.0, 0.18, 0.5, 0.18, 'pants', 'pants', 'pants'), # 0: Left leg
        (0.0, 0.0, 0.0, 0.18, 0.5, 0.18, 'pants', 'pants', 'pants'),   # 1: Right leg
        (-0.19, 0.5, 0.0, 0.38, 0.45, 0.2, 'shirt', 'shirt', 'shirt'), # 2: Torso
        (-0.36, 0.5, 0.0, 0.17, 0.45, 0.18, 'skin', 'skin', 'skin'),   # 3: Left arm
        (0.19, 0.5, 0.0, 0.17, 0.45, 0.18, 'skin', 'skin', 'skin'),    # 4: Right arm
        (-0.2, 0.95, -0.02, 0.4, 0.4, 0.4, 'hair', 'skin', 'skin'),    # 5: Head
        # --- Steve's Eyes ---
        (-0.16, 1.15, 0.38, 0.06, 0.08, 0.02, 'eye_white', 'eye_white', 'eye_white'), # 6: Left eye (White)
        (-0.10, 1.15, 0.38, 0.06, 0.08, 0.02, 'eye_pupil', 'eye_pupil', 'eye_pupil'), # 7: Left eye (Pupil)
        (0.04, 1.15, 0.38, 0.06, 0.08, 0.02, 'eye_pupil', 'eye_pupil', 'eye_pupil'),  # 8: Right eye (Pupil)
        (0.10, 1.15, 0.38, 0.06, 0.08, 0.02, 'eye_white', 'eye_white', 'eye_white'),  # 9: Right eye (White)
    ]

STEVE_PARTS = make_steve_parts()

def part_to_faces(ox, oy, oz, sx, sy, sz, c_top, c_side, c_bottom):
    return [
        (0, 1, 0, [(ox,oy+sy,oz+sz),(ox+sx,oy+sy,oz+sz),(ox+sx,oy+sy,oz),(ox,oy+sy,oz)], c_top, 'top'),
        (0, -1, 0, [(ox,oy,oz),(ox+sx,oy,oz),(ox+sx,oy,oz+sz),(ox,oy,oz+sz)], c_bottom, 'bottom'),
        (1, 0, 0, [(ox+sx,oy,oz+sz),(ox+sx,oy+sy,oz+sz),(ox+sx,oy+sy,oz),(ox+sx,oy,oz)], c_side, '+X'),
        (-1, 0, 0, [(ox,oy,oz),(ox,oy+sy,oz),(ox,oy+sy,oz+sz),(ox,oy,oz+sz)], c_side, '-X'),
        (0, 0, 1, [(ox,oy,oz+sz),(ox,oy+sy,oz+sz),(ox+sx,oy+sy,oz+sz),(ox+sx,oy,oz+sz)], c_side, '+Z'),
        (0, 0, -1, [(ox+sx,oy,oz),(ox+sx,oy+sy,oz),(ox,oy+sy,oz),(ox,oy,oz)], c_side, '-Z'),
    ]

# --- 4. True 3D Rendering Engine ---
def draw_world():
    canvas.delete("all")

    yaw_rad = math.radians(current_angle)
    pitch_rad = math.radians(-30)
    cos_y, sin_y = math.cos(yaw_rad), math.sin(yaw_rad)
    cos_p, sin_p = math.cos(pitch_rad), math.sin(pitch_rad)

    render_list = []

    for (x, y, z), b_type in world.items():
        cx = x + 0.5 - player_x
        cy = y + 0.5
        cz = z + 0.5 - player_z
        rz = cx * sin_y + cz * cos_y
        depth = cy * sin_p + rz * cos_p
        model_key = str(b_type) if str(b_type) in BLOCK_MODELS else str(b_type).split('_')[0]
        render_list.append((depth, x, y, z, model_key, b_type, 'block'))

    for (x, y, z) in clouds_world:
        if (x, y, z) in world: continue
        cx = x + 0.5 - player_x
        cy = y + 0.5
        cz = z + 0.5 - player_z
        rz = cx * sin_y + cz * cos_y
        depth = cy * sin_p + rz * cos_p
        render_list.append((depth, x, y, z, '4', '4', 'block'))

    steve_depth = (player_y * sin_p) + ((0) * cos_p)
    render_list.append((steve_depth - 0.001, None, None, None, None, None, 'steve'))

    render_list.sort(key=lambda b: b[0], reverse=True)

    for depth, x, y, z, model_key, raw_type, kind in render_list:
        if kind == 'steve':
            draw_steve(cos_y, sin_y, cos_p, sin_p)
            continue

        base_tag = f"{x}_{y}_{z}"
        model = BLOCK_MODELS[model_key]

        for (nx, ny, nz, v_list, color_key, click_face) in model:
            if click_face in ADJACENT:
                dx, dy, dz = ADJACENT[click_face]
                adj_pos = (x + dx, y + dy, z + dz)
                adj_type = None
                if adj_pos in world: adj_type = str(world[adj_pos])
                elif adj_pos in clouds_world: adj_type = '4'

                if adj_type:
                    is_me_fluid = str(raw_type).startswith('9')
                    is_adj_fluid = adj_type.startswith('9')
                    is_me_cloud = str(raw_type) == '4'
                    is_adj_cloud = adj_type == '4'
                    is_me_crop = str(raw_type) in ('F', 'W0', 'W1', 'W2')
                    is_adj_crop = adj_type in ('F', 'W0', 'W1', 'W2')

                    if not is_adj_fluid and not is_adj_cloud and not (is_me_crop and is_adj_crop):
                        continue
                    if is_me_fluid and is_adj_fluid:
                        me_lvl = 4 if 'S' in str(raw_type) or '_' not in str(raw_type) else int(str(raw_type).split('_')[1])
                        adj_lvl = 4 if 'S' in adj_type or '_' not in adj_type else int(adj_type.split('_')[1])
                        if adj_lvl >= me_lvl: continue
                    if is_me_cloud and is_adj_cloud:
                        continue

            nrx = nx * cos_y - nz * sin_y
            nrz = nx * sin_y + nz * cos_y
            n_depth = ny * sin_p + nrz * cos_p
            if n_depth > -0.001: continue

            cache_key = (color_key, nx, ny, nz)
            if cache_key not in color_cache:
                dot = nx * SUN_DIR[0] + ny * SUN_DIR[1] + nz * SUN_DIR[2]
                illumination = 0.4 + max(0.0, dot) * 0.6
                color_cache[cache_key] = hex_color(COLORS[color_key], illumination)

            color_str = color_cache[cache_key]

            coords = []
            x_min, x_max, y_min, y_max = 9999, -9999, 9999, -9999

            for (vx, vy, vz) in v_list:
                wx = x + vx - player_x
                wy = y + vy
                wz = z + vz - player_z

                rx = wx * cos_y - wz * sin_y
                rz = wx * sin_y + wz * cos_y
                ry = wy * cos_p - rz * sin_p

                sx = pan_x + rx * SCALE
                sy = pan_y - ry * SCALE
                coords.extend([sx, sy])

                if sx < x_min: x_min = sx
                if sx > x_max: x_max = sx
                if sy < y_min: y_min = sy
                if sy > y_max: y_max = sy

            if x_max < 0 or x_min > 1000 or y_max < 0 or y_min > 650: continue

            outline = "#D0D0D0" if model_key == '4' else "#111111"
            canvas.create_polygon(coords, fill=color_str, outline=outline, tags=(base_tag, click_face))

    canvas.create_text(500, 620, text="Hold Right-Click & Drag to Steer Camera | WASD to Move", 
                       fill="white", font=("Courier", 11, "bold"), tags="ui")

    draw_altimeter()
    draw_dock()
    draw_templates_sidebar()
    draw_quest_panel()

def draw_steve(cos_y, sin_y, cos_p, sin_p):
    facing_rad = math.radians(player_facing)
    fcos, fsin = math.cos(facing_rad), math.sin(facing_rad)
    
    walk_swing = math.sin(steve_walk_cycle) * 0.25

    for idx, (ox, oy, oz, sx, sy, sz, c_top, c_side, c_bottom) in enumerate(STEVE_PARTS):
        my_ox, my_oy, my_oz = ox, oy, oz
        
        # Animate Left Leg
        if idx == 0:
            my_oz += walk_swing
            my_oy += abs(walk_swing) * 0.15 # Lift leg slightly 
        # Animate Right Leg
        elif idx == 1:
            my_oz -= walk_swing
            my_oy += abs(walk_swing) * 0.15 
        # Animate Left Arm (swings opposite to left leg)
        elif idx == 3:
            my_oz -= walk_swing
        # Animate Right Arm (swings opposite to right leg, merges with block hitting)
        elif idx == 4:
            my_oz += walk_swing
            if steve_swing_timer > 0:
                swing_amt = math.sin((steve_swing_timer / 15.0) * math.pi)
                my_oz -= swing_amt * 0.35  
                my_oy += swing_amt * 0.15  

        faces = part_to_faces(my_ox, my_oy, my_oz, sx, sy, sz, c_top, c_side, c_bottom)
        
        for (nx, ny, nz, v_list, color_key, face_name) in faces:
            rnx = nx * fcos - nz * fsin
            rnz = nx * fsin + nz * fcos
            wrx = rnx * cos_y - rnz * sin_y
            wrz = rnx * sin_y + rnz * cos_y
            n_depth = ny * sin_p + wrz * cos_p
            if n_depth > 0.15: continue

            cache_key = ('steve', color_key, round(rnx,2), round(ny,2), round(rnz,2))
            if cache_key not in color_cache:
                dot = rnx * SUN_DIR[0] + ny * SUN_DIR[1] + rnz * SUN_DIR[2]
                illumination = 0.5 + max(0.0, dot) * 0.5
                color_cache[cache_key] = hex_color(COLORS[color_key], illumination)
            color_str = color_cache[cache_key]

            coords = []
            for (vx, vy, vz) in v_list:
                rvx = vx * fcos - vz * fsin
                rvz = vx * fsin + vz * fcos
                wx = player_x + rvx
                wy = player_y + vy
                wz = player_z + rvz
                cx = wx - player_x
                cy = wy
                cz = wz - player_z
                rx = cx * cos_y - cz * sin_y
                rz = cx * sin_y + cz * cos_y
                ry = cy * cos_p - rz * sin_p

                sx_ = pan_x + rx * SCALE
                sy_ = pan_y - ry * SCALE
                coords.extend([sx_, sy_])

            canvas.create_polygon(coords, fill=color_str, outline="#111111", tags=("steve",))

def draw_altimeter():
    alt_x, alt_y, alt_height = 890, 100, 400
    
    # 1 Minecraft block = 1 Meter in real-world scale
    altitude_m = max(0.0, player_y)
    altitude_ft = int(altitude_m * 3.28084)
    
    canvas.create_rectangle(alt_x, alt_y, alt_x + 15, alt_y + alt_height, fill="#222", outline="#444", width=2, tags="ui")
    
    # Visual slider scaling (Set max expected height to 50 meters to provide a realistic bar)
    max_m = 50.0
    fraction = min(1.0, altitude_m / max_m)
    slider_y = (alt_y + alt_height) - (fraction * alt_height)
    
    canvas.create_polygon(alt_x - 12, slider_y, alt_x, slider_y - 8, alt_x, slider_y + 8, fill="#FF3333", outline="black", tags="ui")
    canvas.create_line(alt_x, slider_y, alt_x + 15, slider_y, fill="#FF3333", width=2, tags="ui")
    canvas.create_text(alt_x + 8, alt_y - 15, text="ALT", fill="white", font=("Courier", 12, "bold"), tags="ui")
    
    # Display real-life accurate physics (Meters & Feet)
    canvas.create_text(alt_x - 18, slider_y - 8, text=f"{int(altitude_m)} m", anchor="e", fill="#00FF00", font=("Courier", 10, "bold"), tags="ui")
    canvas.create_text(alt_x - 18, slider_y + 8, text=f"{altitude_ft} ft", anchor="e", fill="white", font=("Courier", 10, "bold"), tags="ui")
    
    mode = "FLYING" if flying else "WALKING"
    canvas.create_text(alt_x + 8, alt_y + alt_height + 20, text=mode, fill="#7CFC00" if flying else "white", font=("Courier", 11, "bold"), tags="ui")

def draw_dock():
    dock_w = 70
    canvas.create_rectangle(0, 0, dock_w, 650, fill="#1a1a1a", outline="#444", tags="ui")
    canvas.create_text(dock_w/2, 20, text="INV", fill="white", font=("Courier", 12, "bold"), tags="ui")
    y_offset = 50
    visual_order = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0', 'P']

    for b_id in visual_order:
        color_key = BLOCK_MODELS[b_id][0][4]
        r, g, b = COLORS[color_key]
        icon_color = f"#{r:02x}{g:02x}{b:02x}"
        outline = "white" if b_id == current_block else "#333"
        width = 3 if b_id == current_block else 1
        canvas.create_rectangle(15, y_offset, 55, y_offset + 40, fill=icon_color, outline=outline, width=width, tags=("ui", "dock", f"block_{b_id}"))
        display_num = "0" if b_id == "0" else b_id
        canvas.create_text(35, y_offset + 20, text=display_num, fill="white", font=("Courier", 14, "bold"), tags=("ui", "dock", f"block_{b_id}"))
        y_offset += 53

def draw_templates_sidebar():
    sidebar_w = 80
    sx0 = 1000 - sidebar_w
    canvas.create_rectangle(sx0, 0, 1000, 650, fill="#1a1a1a", outline="#444", tags="ui")
    canvas.create_text(sx0 + sidebar_w/2, 20, text="PLANS", fill="white", font=("Courier", 12, "bold"), tags="ui")

    templates = [("Hut", "hut"), ("Well", "well"), ("Tree", "tree")]
    y_offset = 50
    for label, t_id in templates:
        canvas.create_rectangle(sx0 + 10, y_offset, sx0 + 70, y_offset + 40, fill="#444", outline="white", tags=("ui", "template", f"tmpl_{t_id}"))
        canvas.create_text(sx0 + 40, y_offset + 20, text=label, fill="white", font=("Courier", 10, "bold"), tags=("ui", "template", f"tmpl_{t_id}"))
        y_offset += 53

def draw_quest_panel():
    px0, py0, pw, ph = 80, 8, 650, 118
    canvas.create_rectangle(px0, py0, px0 + pw, py0 + ph, fill="#111111", outline="#888", width=2, tags="ui")
    canvas.create_text(px0 + 10, py0 + 8, anchor="nw", fill="#FFD700", font=("Courier", 12, "bold"), text="QUEST: Build a Farm & Home", tags="ui")
    yy = py0 + 30
    for flag, label in QUEST_STEPS:
        done = quest_state[flag]
        mark = "[x]" if done else "[ ]"
        color = "#7CFC00" if done else "white"
        canvas.create_text(px0 + 10, yy, anchor="nw", fill=color, font=("Courier", 10), text=f"{mark} {label}", tags="ui")
        yy += 20

# --- 5. Fluid & Cloud Systems ---
def resolve_cloud_block_overlap(cloud_positions):
    resolved = set()
    for pos in cloud_positions:
        if pos not in world:
            resolved.add(pos)
            continue
        x, y, z = pos
        for dx, dy, dz in NEIGHBOR_OFFSETS:
            candidate = (x + dx, y + dy, z + dz)
            if candidate not in world and candidate not in resolved and candidate not in cloud_positions:
                resolved.add(candidate)
                break
    return resolved

def water_tick():
    changes = {}
    for pos, b_type in list(world.items()):
        if str(b_type).startswith('9'):
            parts = str(b_type).split('_')
            is_source = (len(parts) > 1 and parts[1] == 'S')
            level = 4 if is_source else int(parts[1])
            x, y, z = pos
            has_water_above = (x, y+1, z) in world and str(world[(x, y+1, z)]).startswith('9')
            if not is_source and not has_water_above:
                has_stronger_neighbor = False
                for dx, dz in [(1,0), (-1,0), (0,1), (0,-1)]:
                    adj = (x+dx, y, z+dz)
                    if adj in world and str(world[adj]).startswith('9'):
                        adj_p = str(world[adj]).split('_')
                        adj_lvl = 4 if (len(adj_p) > 1 and adj_p[1] == 'S') else int(adj_p[1])
                        if adj_lvl > level:
                            has_stronger_neighbor = True; break
                if not has_stronger_neighbor:
                    if pos not in changes: changes[pos] = None
                    continue

            below = (x, y-1, z)
            if below not in world:
                if y > -15:
                    if below not in changes: changes[below] = '9_4'
            else:
                if str(world[below]).startswith('9'): pass
                elif level > 1:
                    for dx, dz in [(1,0), (-1,0), (0,1), (0,-1)]:
                        adj = (x+dx, y, z+dz)
                        if adj not in world:
                            if adj not in changes: changes[adj] = f'9_{level - 1}'
                        elif str(world[adj]).startswith('9'):
                            adj_p = str(world[adj]).split('_')
                            adj_lvl = 4 if (len(adj_p) > 1 and adj_p[1] == 'S') else int(adj_p[1])
                            if adj_lvl < level - 1: changes[adj] = f'9_{level - 1}'
    if changes:
        for pos, val in changes.items():
            if val is None:
                if pos in world: del world[pos]
            else:
                world[pos] = val
    root.after(80, water_tick)

def cloud_tick():
    global clouds_world
    new_clouds = set()
    for (x, y, z) in clouds_world:
        if x < player_x + 40:
            new_clouds.add((x + 1, y, z))
    if random.random() < 0.2:
        cx = int(player_x) + random.randint(-35, -25)
        cz = int(player_z) + random.randint(-15, GRID_SIZE + 15)
        cy = random.randint(20, 25)
        for dx in range(random.randint(3, 7)):
            for dz in range(random.randint(3, 7)):
                if random.random() > 0.35:
                    new_clouds.add((cx + dx, cy, cz + dz))
    clouds_world = resolve_cloud_block_overlap(new_clouds)
    root.after(1000, cloud_tick)

def crop_tick():
    changes = {}
    for pos, b_type in list(world.items()):
        if b_type == 'W0' and random.random() < 0.15: changes[pos] = 'W1'
        elif b_type == 'W1' and random.random() < 0.15: changes[pos] = 'W2'
    for pos, val in changes.items(): world[pos] = val
    root.after(2500, crop_tick)

# --- 6. Quest helpers ---
def check_house_progress():
    if quest_state["house_done"]: return
    plank_positions = [pos for pos, b in world.items() if b == 'P']
    if len(plank_positions) < 5: return
    ys = [p[1] for p in plank_positions]
    if len(set(ys)) >= 2:
        quest_state["house_done"] = True

def advance_quest_from_action(action):
    global harvested_wheat
    if action == "till" and not quest_state["till_done"]: quest_state["till_done"] = True
    elif action == "plant" and not quest_state["plant_done"]: quest_state["plant_done"] = True
    elif action == "harvest":
        harvested_wheat += 1
        if not quest_state["harvest_done"]: quest_state["harvest_done"] = True

# --- 7. Player movement, jump/fly, farming actions ---
def is_solid_at(x, y, z):
    key = (int(math.floor(x)), int(math.floor(y)), int(math.floor(z)))
    if key not in world: return False
    b = world[key]
    if b in ('F', 'W0', 'W1', 'W2') or str(b).startswith('9'): return False
    return True

def can_move_to(x, y, z):
    if is_solid_at(x, y + 0.1, z): return False 
    if is_solid_at(x, y + 0.7, z): return False 
    if is_solid_at(x, y + 1.3, z): return False 
    return True

def ground_height(x, z):
    xi, zi = int(math.floor(x)), int(math.floor(z))
    highest = 0
    for (bx, by, bz), b in world.items():
        if bx == xi and bz == zi and not (b in ('F','W0','W1','W2') or str(b).startswith('9')):
            if by + 1 > highest:
                highest = by + 1
    return highest

def player_tick():
    global player_x, player_z, player_y, player_vy, current_angle, target_angle
    global steve_swing_timer, steve_walk_cycle, player_facing, mouse_x, mouse_y

    if steve_swing_timer > 0: steve_swing_timer -= 1

    # --- AIM STEVE AT MOUSE CURSOR ---
    pitch_rad = math.radians(-30)
    cos_p, sin_p = math.cos(pitch_rad), math.sin(pitch_rad)
    
    steve_screen_x = pan_x
    steve_screen_y = pan_y - (player_y * cos_p) * SCALE
    
    dx_screen = mouse_x - steve_screen_x
    dy_screen = mouse_y - steve_screen_y
    
    rx = dx_screen / SCALE
    rz = dy_screen / (sin_p * SCALE) if sin_p != 0 else 0
    
    yaw_rad = math.radians(current_angle)
    cos_y, sin_y = math.cos(yaw_rad), math.sin(yaw_rad)
    
    cx = rx * cos_y + rz * sin_y
    cz = -rx * sin_y + rz * cos_y
    
    target_facing = math.degrees(math.atan2(-cx, cz))
    
    diff = (target_facing - player_facing) % 360
    if diff > 180: diff -= 360
    player_facing += diff * 0.4  

    # --- INDEPENDENT MOVEMENT ---
    move_fwd = 0
    strafe = 0
    if "w" in keys_held: move_fwd += 1
    if "s" in keys_held: move_fwd -= 1
    if "d" in keys_held: strafe += 1
    if "a" in keys_held: strafe -= 1

    if move_fwd or strafe:
        cam_rad = math.radians(current_angle)
        fcos, fsin = math.cos(cam_rad), math.sin(cam_rad)

        length = math.hypot(strafe, move_fwd)
        mf = move_fwd / length
        st = strafe / length

        dx = (fsin * mf + fcos * st) * MOVE_SPEED
        dz = (fcos * mf - fsin * st) * MOVE_SPEED

        new_x = player_x + dx
        new_z = player_z + dz
        
        if can_move_to(new_x, player_y, player_z): player_x = new_x
        if can_move_to(player_x, player_y, new_z): player_z = new_z

        steve_walk_cycle += 0.3
    else:
        steve_walk_cycle *= 0.8
        if abs(steve_walk_cycle) < 0.01:
            steve_walk_cycle = 0.0

    ground = ground_height(player_x, player_z)

    if flying:
        player_vy = 0.0
        if "space" in keys_held: player_y += FLY_SPEED
        if "shift_l" in keys_held or "shift_r" in keys_held: player_y -= FLY_SPEED
        if player_y < ground: player_y = ground
    else:
        player_vy -= GRAVITY
        player_y += player_vy
        if player_y <= ground:
            player_y = ground
            player_vy = 0.0

    draw_world()
    root.after(30, player_tick)

def try_jump_or_fly():
    global last_space_time, flying, player_vy
    now_t = _time_now()
    if last_space_time is not None and (now_t - last_space_time) < DOUBLE_TAP_WINDOW:
        flying = not flying
        player_vy = 0.0
        last_space_time = None
    else:
        last_space_time = now_t
        if not flying and abs(player_y - ground_height(player_x, player_z)) < 0.05:
            player_vy = JUMP_VZ

def _time_now():
    return time.time()

def facing_target_block():
    rad = math.radians(player_facing)
    tx = player_x + math.sin(rad) * 0.9
    tz = player_z + math.cos(rad) * 0.9
    ty = ground_height(tx, tz) - 1
    return (int(math.floor(tx)), max(0, int(ty)), int(math.floor(tz)))

def do_till(event=None):
    global steve_swing_timer
    steve_swing_timer = 15
    pos = facing_target_block()
    below = (pos[0], pos[1], pos[2])
    if below in world and world[below] == '1':
        world[below] = 'F'
        advance_quest_from_action("till")
        draw_world()

def do_plant_or_harvest(event=None):
    global steve_swing_timer
    steve_swing_timer = 15
    pos = facing_target_block()
    if pos in world:
        b = world[pos]
        if b == 'F':
            world[pos] = 'W0'
            advance_quest_from_action("plant")
            draw_world()
        elif b == 'W2':
            world[pos] = 'F'
            advance_quest_from_action("harvest")
            draw_world()

# --- 8. Building Interactions & UI Clicks ---
def build_template(t_id):
    global steve_swing_timer
    steve_swing_timer = 15
    rad = math.radians(player_facing)
    
    tx = int(math.floor(player_x + math.sin(rad) * 3.0))
    tz = int(math.floor(player_z + math.cos(rad) * 3.0))
    ty = ground_height(tx, tz)

    if t_id == "hut":
        for dx in range(-1, 2):
            for dz in range(-1, 2):
                world[(tx+dx, ty, tz+dz)] = 'P' 
                world[(tx+dx, ty+3, tz+dz)] = '5' 
                if dx != 0 or dz != 0:
                    if not (dx == 0 and dz == 1):
                        world[(tx+dx, ty+1, tz+dz)] = 'P'
                        world[(tx+dx, ty+2, tz+dz)] = 'P'
    elif t_id == "well":
        for dx in range(-1, 2):
            for dz in range(-1, 2):
                if dx == 0 and dz == 0: world[(tx, ty, tz)] = '9_S' 
                else: world[(tx+dx, ty, tz+dz)] = '2' 
        world[(tx-1, ty+1, tz)] = '5'
        world[(tx-1, ty+2, tz)] = '5'
        world[(tx+1, ty+1, tz)] = '5'
        world[(tx+1, ty+2, tz)] = '5'
        for dx in range(-1, 2): world[(tx+dx, ty+3, tz)] = '2'
    elif t_id == "tree":
        for h in range(1, 4): world[(tx, ty+h-1, tz)] = '5'
        for dx in range(-1, 2):
            for dz in range(-1, 2):
                world[(tx+dx, ty+3, tz+dz)] = '6'
        world[(tx, ty+4, tz)] = '6'

    check_house_progress()
    draw_world()

is_left_dragging = False
def on_left_press(event): global is_left_dragging; is_left_dragging = False
def on_left_drag(event): 
    global is_left_dragging; is_left_dragging = True
    update_mouse_pos(event)

def update_mouse_pos(event):
    global mouse_x, mouse_y
    mouse_x = event.x
    mouse_y = event.y

canvas.bind("<Motion>", update_mouse_pos)

def handle_build(event):
    global clouds_world, steve_swing_timer
    if is_left_dragging: return
    item = canvas.find_withtag("current")
    if not item: return
    tags = canvas.gettags(item[0])

    if "dock" in tags:
        for t in tags:
            if t.startswith("block_"): set_block(t.split("_")[1])
        return

    if "template" in tags:
        for t in tags:
            if t.startswith("tmpl_"): build_template(t.split("_")[1])
        return

    if "ui" in tags or len(tags) < 2: return

    steve_swing_timer = 15
    coords = tags[0].split('_')
    x, y, z = int(coords[0]), int(coords[1]), int(coords[2])
    face = tags[1]

    if face in ADJACENT:
        dx, dy, dz = ADJACENT[face]
        new_block = '9_S' if current_block == '9' else current_block
        world[(x + dx, y + dy, z + dz)] = new_block
        if clouds_world: clouds_world = resolve_cloud_block_overlap(clouds_world)
        check_house_progress()
        draw_world()

def handle_destroy(event):
    global steve_swing_timer
    if is_dragging: return
    item = canvas.find_withtag("current")
    if not item or "ui" in canvas.gettags(item[0]): return

    tags = canvas.gettags(item[0])
    if len(tags) < 2: return
    
    steve_swing_timer = 15
    coords = tags[0].split('_')
    x, y, z = int(coords[0]), int(coords[1]), int(coords[2])

    if (x, y, z) in world:
        del world[(x, y, z)]
        draw_world()

# --- 9. Controls & Panning ---
def pan_view(dx, dy):
    global pan_x, pan_y; pan_x += dx; pan_y += dy; draw_world()

canvas.bind("<MouseWheel>", lambda e: pan_view(0, e.delta // 12 if sys.platform == "win32" else e.delta))

def trackpad_free_rotate(event):
    global target_angle
    delta = event.delta // 10 if sys.platform == "win32" else event.delta
    target_angle += delta * 0.5

canvas.bind("<Shift-MouseWheel>", trackpad_free_rotate)

drag_start_x = 0; is_dragging = False; drag_dist = 0
def on_right_press(event):
    global drag_start_x, is_dragging, drag_dist
    item = canvas.find_withtag("current")
    if item and "ui" in canvas.gettags(item[0]): return
    drag_start_x = event.x; is_dragging = False; drag_dist = 0

def on_right_drag(event):
    global drag_start_x, is_dragging, target_angle, drag_dist
    update_mouse_pos(event)
    diff = event.x - drag_start_x
    drag_dist += abs(diff)
    if drag_dist > 3: 
        is_dragging = True
        target_angle -= diff * 0.4
        drag_start_x = event.x

def on_right_release(event):
    if not is_dragging: handle_destroy(event)

canvas.bind("<ButtonPress-1>", on_left_press)
canvas.bind("<B1-Motion>", on_left_drag)
canvas.bind("<ButtonRelease-1>", handle_build)

canvas.bind("<ButtonPress-2>", on_right_press)
canvas.bind("<B2-Motion>", on_right_drag)
canvas.bind("<ButtonRelease-2>", on_right_release)
canvas.bind("<ButtonPress-3>", on_right_press)
canvas.bind("<B3-Motion>", on_right_drag)
canvas.bind("<ButtonRelease-3>", on_right_release)

def on_key_down(event):
    key = event.keysym.lower()
    if key == "space" and "space" not in keys_held: try_jump_or_fly()
    keys_held.add(key)

def on_key_up(event): keys_held.discard(event.keysym.lower())

root.bind("<KeyPress>", on_key_down)
root.bind("<KeyRelease>", on_key_up)

def turn_camera(delta):
    global target_angle
    target_angle += delta

root.bind("q", lambda e: turn_camera(15))
root.bind("e", lambda e: turn_camera(-15))
root.bind("f", do_till)
root.bind("g", do_plant_or_harvest)

# --- 10. Interface Setup ---
def update_title():
    root.title(f"Steve | Holding: {BLOCK_NAMES.get(current_block,'?')} | "
               f"{'Flying' if flying else 'Walking'} | Wheat harvested: {harvested_wheat}")

def set_block(b):
    global current_block; current_block = b
    draw_world()
    update_title()

for key in BLOCK_NAMES.keys(): root.bind(key, lambda e, b=key: set_block(b))
root.bind("f", do_till)
root.bind("g", do_plant_or_harvest)

# --- 11. Animation / Title ticking ---
def animation_loop():
    global current_angle
    diff = (target_angle - current_angle) % 360
    if diff > 180: diff -= 360

    if abs(diff) > 0.1: current_angle += diff * 0.2
    else: current_angle = target_angle

    update_title()
    root.after(16, animation_loop)

# --- 12. Generate a bigger, varied World ---
def generate_world():
    rng = random.Random(1234)
    for x in range(GRID_SIZE):
        for z in range(GRID_SIZE):
            world[(x, 0, z)] = '1'
            if rng.random() < 0.02: world[(x, 1, z)] = '2'
    for x in range(6, 10):
        for z in range(6, 10): world[(x, 0, z)] = '9_S'
    for _ in range(10):
        tx, tz = rng.randint(2, GRID_SIZE - 3), rng.randint(2, GRID_SIZE - 3)
        if (tx, 0, tz) in world and world[(tx, 0, tz)] == '1':
            for h in range(1, 4): world[(tx, h, tz)] = '5'
            world[(tx, 4, tz)] = '6'

generate_world()

for _ in range(12):
    cx = int(player_x) + random.randint(-20, 30)
    cz = int(player_z) + random.randint(-15, GRID_SIZE + 15)
    cy = random.randint(20, 25)
    for dx in range(random.randint(3, 6)):
        for dz in range(random.randint(3, 6)):
            if random.random() > 0.35: clouds_world.add((cx + dx, cy, cz + dz))

clouds_world = resolve_cloud_block_overlap(clouds_world)

set_block('1')
draw_world()
animation_loop()
water_tick()
cloud_tick()
crop_tick()
player_tick()
root.mainloop()

```
