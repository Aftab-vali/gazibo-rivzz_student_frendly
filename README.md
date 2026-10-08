# gazibo-rivzz_student_frendly
# 🤖 ROS 2 Humble · TurtleBot3 · Gazebo · RViz2 — Student Guide

Build a simulated TurtleBot3 robot that drives around an indoor world while you watch its **LiDAR, odometry, map and TF frames** live in RViz2. Later you can add **SLAM** (mapping) and **Nav2** (autonomous navigation).

> ⏱️ **Time:** about 15–25 minutes (mostly downloads). **Everyone who follows these steps gets the same result.**

---

## 📦 How this guide works

This is **one single file**. You do not download anything else. You will:

1. **Paste one block** into the terminal → it creates the setup script.
2. **Paste one block** into the terminal → it creates the check script.
3. Run `setup` once, then `check` after starting Gazebo.

| Script (created by you in Step 2) | What it does |
| --- | --- |
| `setup_turtlebot3_sim.sh` | **One command** that installs and configures everything |
| `check_simulation.sh` | **One command** that proves the simulation really works (it even drives the robot for you) |

---

## 1️⃣ Requirements

| Item | Needed |
| --- | --- |
| OS | **Ubuntu 22.04** (native) or **WSL2 + Ubuntu 22.04** on Windows 10/11 |
| Disk | ~10 GB free |
| RAM | 8 GB minimum |
| Internet | During installation only |
| Account | A normal user with `sudo` rights |

### 🪟 Windows / WSL2 users — do this first (3 minutes)

1. In **Windows PowerShell** update WSL, then restart it:
   ```powershell
   wsl --update
   wsl --shutdown
   ```
2. *(Recommended)* give WSL more memory. Create `C:\Users\<YourName>\.wslconfig`:
   ```ini
   [wsl2]
   memory=8GB
   processors=4
   ```
   then run `wsl --shutdown` again.
3. Open **Ubuntu 22.04** (or VS Code → *WSL: Ubuntu-22.04*). All commands below run **inside Ubuntu**.
4. Keep your files in the Linux home (`~`), **not** in `/mnt/c/...` (very slow).

---

## 2️⃣ Install — one command

### Step A — create a working folder

```bash
mkdir -p ~/tb3_sim && cd ~/tb3_sim
```

### Step B — create the setup script (copy → paste → Enter)

Click the copy button of the block below, paste it in the Ubuntu terminal and press **Enter**. Nothing is installed yet; it only creates the file `setup_turtlebot3_sim.sh`.

<details>
<summary><b>👉 Click to open: setup script block</b></summary>

```bash
cat > ~/tb3_sim/setup_turtlebot3_sim.sh << 'TB3_SETUP_END'
#!/usr/bin/env bash
# =============================================================================
#  TurtleBot3 + Gazebo + RViz2 : one-shot setup
#  Target : Ubuntu 22.04 (native or WSL2)  +  ROS 2 Humble
#  Usage  : bash setup_turtlebot3_sim.sh          (do NOT use sudo)
#  Safe to run again: every step checks what already exists.
# =============================================================================
set -eo pipefail

BEGIN_MARK="#TB3SIM-BEGIN"
END_MARK="#TB3SIM-END"
SYS_MODELS="${SYS_MODELS:-/usr/share/gazebo-11/models}"
USER_MODELS="${USER_MODELS:-$HOME/.gazebo/models}"
BASHRC="${BASHRC:-$HOME/.bashrc}"

say()  { printf '\n\033[1;36m==> %s\033[0m\n' "$*"; }
ok()   { printf '\033[1;32m  ✔ %s\033[0m\n' "$*"; }
warn() { printf '\033[1;33m  ! %s\033[0m\n' "$*"; }
die()  { printf '\033[1;31m  ✘ %s\033[0m\n' "$*" >&2; exit 1; }

# -----------------------------------------------------------------------------
check_system() {
  say "Step 1/5  Checking your system"
  [ "$(id -u)" -ne 0 ] || die "Do not run this with sudo. Run:  bash setup_turtlebot3_sim.sh"

  . /etc/os-release
  [ "${VERSION_ID:-}" = "22.04" ] || die "This guide needs Ubuntu 22.04 (you have ${PRETTY_NAME:-unknown})."
  ok "Ubuntu 22.04"

  if grep -qi microsoft /proc/version 2>/dev/null; then
    IS_WSL=1; ok "Running inside WSL2"
  else
    IS_WSL=0; ok "Native Ubuntu"
  fi

  case "$(pwd)" in
    /mnt/*) warn "You are working inside /mnt/... (Windows disk). This is slow. Copy this folder to your Linux home (~) later." ;;
  esac

  echo "  (sudo may ask for your Ubuntu password – nothing shows while you type)"
  sudo -v || die "sudo failed."
}

# -----------------------------------------------------------------------------
install_ros_if_missing() {
  say "Step 2/5  ROS 2 Humble"
  if [ -f /opt/ros/humble/setup.bash ]; then
    ok "ROS 2 Humble already installed"
    return
  fi
  warn "ROS 2 Humble was not found."
  read -r -p "  Install ros-humble-desktop now? (about 3 GB download) [y/N] " ans
  [ "$ans" = "y" ] || [ "$ans" = "Y" ] || die "Install ROS 2 Humble first: https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html"

  sudo apt update
  sudo apt install -y locales software-properties-common curl
  sudo locale-gen en_US en_US.UTF-8
  sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
  sudo add-apt-repository universe -y
  sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
       -o /usr/share/keyrings/ros-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo "$UBUNTU_CODENAME") main" \
       | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
  sudo apt update
  sudo apt install -y ros-humble-desktop
  ok "ROS 2 Humble installed"
}

# -----------------------------------------------------------------------------
install_packages() {
  say "Step 3/5  Installing Gazebo, TurtleBot3, SLAM and Nav2 (apt – no compiling)"
  sudo apt update
  sudo apt install -y \
    psmisc mesa-utils \
    python3-colcon-common-extensions \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-turtlebot3 \
    ros-humble-turtlebot3-simulations \
    ros-humble-turtlebot3-gazebo \
    ros-humble-turtlebot3-teleop \
    ros-humble-turtlebot3-bringup \
    ros-humble-turtlebot3-navigation2 \
    ros-humble-slam-toolbox \
    ros-humble-navigation2 \
    ros-humble-nav2-bringup \
    ros-humble-tf2-tools
  ok "Packages installed"
}

# -----------------------------------------------------------------------------
# Gazebo needs a floor (ground_plane) and a light (sun). If they cannot be found,
# the robot falls through the world forever. This guarantees both exist, offline.
install_local_models() {
  say "Step 4/5  Making sure Gazebo has a floor and a light"

  if [ -f "$SYS_MODELS/ground_plane/model.sdf" ]; then
    ok "ground_plane found in $SYS_MODELS"
  else
    warn "ground_plane missing in $SYS_MODELS – creating a local copy"
    mkdir -p "$USER_MODELS/ground_plane"
    cat > "$USER_MODELS/ground_plane/model.config" << 'EOF'
<?xml version="1.0"?>
<model>
  <name>Ground Plane</name>
  <version>1.0</version>
  <sdf version="1.6">model.sdf</sdf>
  <description>Simple ground plane</description>
</model>
EOF
    cat > "$USER_MODELS/ground_plane/model.sdf" << 'EOF'
<?xml version="1.0"?>
<sdf version="1.6">
  <model name="ground_plane">
    <static>true</static>
    <link name="link">
      <collision name="collision">
        <geometry><plane><normal>0 0 1</normal><size>100 100</size></plane></geometry>
        <surface><friction><ode><mu>100</mu><mu2>50</mu2></ode></friction></surface>
      </collision>
      <visual name="visual">
        <cast_shadows>false</cast_shadows>
        <geometry><plane><normal>0 0 1</normal><size>100 100</size></plane></geometry>
        <material>
          <script>
            <uri>file://media/materials/scripts/gazebo.material</uri>
            <name>Gazebo/Grey</name>
          </script>
        </material>
      </visual>
    </link>
  </model>
</sdf>
EOF
    ok "Created $USER_MODELS/ground_plane"
  fi

  if [ -f "$SYS_MODELS/sun/model.sdf" ]; then
    ok "sun found in $SYS_MODELS"
  else
    warn "sun missing in $SYS_MODELS – creating a local copy"
    mkdir -p "$USER_MODELS/sun"
    cat > "$USER_MODELS/sun/model.config" << 'EOF'
<?xml version="1.0"?>
<model>
  <name>Sun</name>
  <version>1.0</version>
  <sdf version="1.6">model.sdf</sdf>
  <description>Directional light</description>
</model>
EOF
    cat > "$USER_MODELS/sun/model.sdf" << 'EOF'
<?xml version="1.0"?>
<sdf version="1.6">
  <light type="directional" name="sun">
    <cast_shadows>true</cast_shadows>
    <pose>0 0 10 0 0 0</pose>
    <diffuse>0.8 0.8 0.8 1</diffuse>
    <specular>0.2 0.2 0.2 1</specular>
    <attenuation>
      <range>1000</range>
      <constant>0.9</constant>
      <linear>0.01</linear>
      <quadratic>0.001</quadratic>
    </attenuation>
    <direction>-0.5 0.1 -0.9</direction>
  </light>
</sdf>
EOF
    ok "Created $USER_MODELS/sun"
  fi
}

# -----------------------------------------------------------------------------
write_bashrc() {
  say "Step 5/5  Saving environment settings to ~/.bashrc"
  touch "$BASHRC"
  # remove any previous block written by this script (keeps the script re-runnable)
  sed -i "/^${BEGIN_MARK}\$/,/^${END_MARK}\$/d" "$BASHRC"

  {
    echo "$BEGIN_MARK"
    echo "# TurtleBot3 simulation environment (managed by setup_turtlebot3_sim.sh)"
    echo 'source /opt/ros/humble/setup.bash'
    echo 'export TURTLEBOT3_MODEL=burger'
    echo '# Gazebo built-in model/resource/plugin paths (floor, sun, materials)'
    echo '[ -f /usr/share/gazebo/setup.sh ] && source /usr/share/gazebo/setup.sh'
    echo '# Must come AFTER setup.sh: never try to download models from the internet'
    echo 'export GAZEBO_MODEL_DATABASE_URI=""'
    echo "export GAZEBO_MODEL_PATH=\"\${GAZEBO_MODEL_PATH:+\$GAZEBO_MODEL_PATH:}$USER_MODELS\""
    if [ "${IS_WSL:-0}" = "1" ]; then
      echo '# WSL2: software rendering avoids a black Gazebo window'
      echo 'export LIBGL_ALWAYS_SOFTWARE=1'
    fi
    echo "$END_MARK"
  } >> "$BASHRC"
  ok "Settings written to $BASHRC"
}

# -----------------------------------------------------------------------------
verify() {
  say "Final check"
  if bash -c 'source /opt/ros/humble/setup.bash && ros2 pkg prefix turtlebot3_gazebo >/dev/null 2>&1'; then
    ok "turtlebot3_gazebo is installed"
  else
    die "turtlebot3_gazebo was not found. Scroll up for apt errors."
  fi
  bash -c 'source /opt/ros/humble/setup.bash && ros2 pkg prefix slam_toolbox >/dev/null 2>&1' && ok "slam_toolbox is installed"
  bash -c 'source /opt/ros/humble/setup.bash && ros2 pkg prefix nav2_bringup >/dev/null 2>&1' && ok "nav2_bringup is installed"

  cat << 'EOF'

--------------------------------------------------------------------
  SETUP COMPLETE 🎉

  NEXT:  close ALL terminals, open a NEW one, and run

      bash check_simulation.sh      (after Gazebo is running – see README)

  Start the simulation with:

      killall -9 gzserver gzclient 2>/dev/null
      ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
--------------------------------------------------------------------
EOF
}

main() {
  check_system
  install_ros_if_missing
  install_packages
  install_local_models
  write_bashrc
  verify
}

# run main only when executed (not when sourced for testing)
if [ "${BASH_SOURCE[0]}" = "$0" ]; then
  main "$@"
fi
TB3_SETUP_END
chmod +x ~/tb3_sim/setup_turtlebot3_sim.sh
echo "Created ~/tb3_sim/setup_turtlebot3_sim.sh"
```

</details>

### Step C — create the check script (copy → paste → Enter)

<details>
<summary><b>👉 Click to open: check script block</b></summary>

```bash
cat > ~/tb3_sim/check_simulation.sh << 'TB3_CHECK_END'
#!/usr/bin/env bash
# =============================================================================
#  Checks that the TurtleBot3 simulation is REALLY working.
#  Run it in a NEW terminal while Gazebo (Terminal 1) is running:
#       bash check_simulation.sh
#  It drives the robot ~40 cm forward and back automatically.
# =============================================================================
set +e
source /opt/ros/humble/setup.bash 2>/dev/null || { echo "ROS 2 Humble not found. Run setup_turtlebot3_sim.sh first."; exit 1; }

PASS=0; FAIL=0
good() { printf '\033[1;32m  PASS\033[0m  %s\n' "$*"; PASS=$((PASS+1)); }
bad()  { printf '\033[1;31m  FAIL\033[0m  %s\n' "$*"; FAIL=$((FAIL+1)); }
hint() { printf '        \033[0;33m→ %s\033[0m\n' "$*"; }

get_pos() {  # $1 = x | y | z
  timeout 8 ros2 topic echo /odom --once --field "pose.pose.position.$1" 2>/dev/null | head -n1
}

echo
echo "TurtleBot3 simulation check"
echo "---------------------------"

# 1. Gazebo server
if pgrep -x gzserver >/dev/null; then
  good "Gazebo server (gzserver) is running"
else
  bad "Gazebo is not running"
  hint "Terminal 1:  killall -9 gzserver gzclient 2>/dev/null; ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py"
  echo; echo "Result: $PASS passed, $FAIL failed"; exit 1
fi

# 2. Topics
TOPICS=$(timeout 8 ros2 topic list 2>/dev/null)
for t in /cmd_vel /odom /scan /tf; do
  if echo "$TOPICS" | grep -qx "$t"; then good "Topic $t exists"; else bad "Topic $t is missing"; hint "The robot was not spawned. Restart Gazebo (see README → Troubleshooting)."; fi
done

# 3. LiDAR data
if timeout 8 ros2 topic echo /scan --once --qos-reliability best_effort >/dev/null 2>&1; then
  good "LiDAR (/scan) is publishing data"
else
  bad "No LiDAR data on /scan"
fi

# 4. Floor test (robot must NOT be falling)
Z=$(get_pos z)
if [ -z "$Z" ]; then
  bad "No odometry data on /odom"
  Z=""
elif awk -v z="$Z" 'BEGIN{exit !(z>-0.5 && z<0.5)}'; then
  good "Robot is standing on the floor (z = $Z)"
else
  bad "Robot is FALLING through the world (z = $Z)"
  hint "Gazebo has no floor. Re-run: bash setup_turtlebot3_sim.sh   then restart Gazebo."
fi

# 5. Movement test (only if the robot stands on the floor)
if [ -n "$Z" ] && awk -v z="$Z" 'BEGIN{exit !(z>-0.5 && z<0.5)}'; then
  X0=$(get_pos x)
  echo "  ...driving forward for 3 seconds"
  timeout 3 ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.15}, angular: {z: 0.0}}" >/dev/null 2>&1
  ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0}, angular: {z: 0.0}}" >/dev/null 2>&1
  sleep 1
  X1=$(get_pos x)
  if [ -n "$X0" ] && [ -n "$X1" ] && awk -v a="$X0" -v b="$X1" 'BEGIN{d=b-a; if(d<0)d=-d; exit !(d>0.1)}'; then
    good "Robot moved (x: $X0 → $X1)"
  else
    bad "Robot did not move (x: $X0 → $X1)"
    hint "Is Gazebo paused? Press the ▶ button at the bottom of the Gazebo window."
  fi
  echo "  ...driving back"
  timeout 3 ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: -0.15}, angular: {z: 0.0}}" >/dev/null 2>&1
  ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0}, angular: {z: 0.0}}" >/dev/null 2>&1
fi

echo "---------------------------"
if [ "$FAIL" -eq 0 ]; then
  printf '\033[1;32mResult: ALL %d CHECKS PASSED 🎉  Your simulation works.\033[0m\n' "$PASS"
  exit 0
else
  printf '\033[1;31mResult: %d passed, %d FAILED.  Read the → hints above, or README → Troubleshooting.\033[0m\n' "$PASS" "$FAIL"
  exit 1
fi
TB3_CHECK_END
chmod +x ~/tb3_sim/check_simulation.sh
echo "Created ~/tb3_sim/check_simulation.sh"
```

</details>

Confirm both files exist:

```bash
ls ~/tb3_sim        # check_simulation.sh  setup_turtlebot3_sim.sh
```

### Step D — run the setup script

```bash
cd ~/tb3_sim
bash setup_turtlebot3_sim.sh
```

- Enter your Ubuntu password if asked (nothing is shown while typing).
- If ROS 2 Humble is missing, the script offers to install it. Answer `y`.
- Wait for **`SETUP COMPLETE 🎉`**.

### Step E — close ALL terminals, then open a new one

The new settings are only loaded by **new** terminals. Check:

```bash
echo $ROS_DISTRO          # humble
echo $TURTLEBOT3_MODEL    # burger
```

### 🔍 What the setup script does (so nothing is magic)

1. Checks Ubuntu 22.04 and detects WSL2.
2. Installs ROS 2 Humble if you agree.
3. Installs by `apt`: Gazebo-ROS, TurtleBot3 (simulation, teleop, bringup, navigation2), SLAM Toolbox, Nav2, TF tools. **No compiling needed.**
4. Makes sure Gazebo has a **floor (`ground_plane`) and a light (`sun`)**. If they cannot be found, it creates them offline in `~/.gazebo/models`. *(Without a floor the robot falls forever and you see nothing.)*
5. Adds this block to `~/.bashrc` (you can re-run the script safely, the block is replaced, not duplicated):
   ```bash
   source /opt/ros/humble/setup.bash
   export TURTLEBOT3_MODEL=burger
   source /usr/share/gazebo/setup.sh              # Gazebo model/resource paths
   export GAZEBO_MODEL_DATABASE_URI=""            # never download models (prevents long hangs)
   export GAZEBO_MODEL_PATH=...:$HOME/.gazebo/models
   export LIBGL_ALWAYS_SOFTWARE=1                 # WSL2 only: avoids a black Gazebo window
   ```

---

## 3️⃣ Run the simulation (3 terminals)

Open **3 terminals** (in VS Code use the **+** button). Start them **in this order** and wait for each ✅ before continuing.

```text
Terminal 1 → Gazebo      Terminal 2 → RViz2      Terminal 3 → Keyboard teleop
       └────────── ROS 2 topics: /scan  /odom  /tf  /cmd_vel ──────────┘
```

### 🖥️ Terminal 1 — Gazebo

```bash
killall -9 gzserver gzclient 2>/dev/null
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

✅ **Ready when**

- the log shows `Publishing odom transforms between [odom] and [base_footprint]`, and
- the Gazebo window shows a hexagon world with a 3×3 grid of pillars, **with shadows and shading**.

> ⏳ The **first** launch can take 1–3 minutes (Gazebo builds its cache). **Do not press Ctrl+C.** Later launches take seconds.
> The robot is only **14 cm wide**, so from above it looks like a tiny dark dot near the pillars (it starts at x = −2 m, y = −0.5 m).

**Messages you can ignore:** `Gazebo reaches end-of-life…`, `stereo is NOT SUPPORTED`, `process has finished cleanly`.

### ✅ Run the automatic check (new terminal)

```bash
cd ~/tb3_sim
bash check_simulation.sh
```

You should see all **PASS** and `ALL … CHECKS PASSED 🎉`. The robot drives forward and back by itself. **If anything says FAIL, read the → hint under it and go to [Troubleshooting](#-troubleshooting).** Do not continue until everything passes.

### 👀 Terminal 2 — RViz2

```bash
ros2 launch turtlebot3_bringup rviz2.launch.py
```

RViz2 opens **zoomed far out** and the LiDAR dots are tiny. Do this **first-time setup**:

| # | Where | Change |
| --- | --- | --- |
| 1 | **Global Options → Fixed Frame** | `odom` |
| 2 | **Mouse scroll wheel** over the 3D view | zoom in until the grid squares are big |
| 3 | **Views panel → Target Frame** | `base_footprint` (camera follows the robot) |
| 4 | **LaserScan → Size (m)** | `0.01` → `0.05` |
| 5 | **Add → By display type → RobotModel** (if missing) | Description Topic `/robot_description` |
| 6 | **Add → By display type → TF** (optional) | shows coordinate frames |

LaserScan must use topic `/scan` with **Reliability Policy: Best Effort**.

✅ **Ready when:** you see the robot body and a ring of red/pink LiDAR dots.

### ⌨️ Terminal 3 — Keyboard teleop

```bash
ros2 run turtlebot3_teleop teleop_keyboard
```

Click inside this terminal, then press:

| Key | Action |
| --- | --- |
| `w` | forward speed **+0.01 m/s** per press |
| `x` | backward speed |
| `a` / `d` | turn left / right |
| `s` or `space` | **STOP** |

> ⚠️ Each press changes the speed only a little. **Press `w` about 10–15 times** (≈0.15 m/s) before expecting to see motion. Maximum speed is 0.22 m/s. The numbers in the terminal only show the *command*, not that the robot moved — use `check_simulation.sh` to prove it.

### 🎉 Success

- The robot moves in **Gazebo**
- The robot and LiDAR move in **RViz2**
- `bash check_simulation.sh` prints **ALL CHECKS PASSED**

---

## 4️⃣ Understand what you are seeing

```text
Gazebo (physics + simulated robot)  ──/scan /odom /tf──►  ROS 2  ──►  RViz2 (visualisation)
          ▲                                                    │
          └──────────────── /cmd_vel ◄──── teleop / Nav2 ◄─────┘
```

| Tool | Question it answers |
| --- | --- |
| **Gazebo** | *What is physically happening?* |
| **RViz2** | *What does ROS think is happening?* |

**Key topics**

```bash
ros2 topic list
ros2 topic hz /scan                      # LiDAR rate (~5 Hz)
ros2 topic echo /odom --once             # robot position
ros2 topic info /cmd_vel                 # who publishes / subscribes
```

**`/cmd_vel` meaning:** `linear.x` = forward/back, `angular.z` = turn left/right. TurtleBot3 cannot move sideways.

**Drive without the keyboard** (stop with Ctrl+C, then send the stop command):

```bash
ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.15}, angular: {z: 0.0}}"
ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0}, angular: {z: 0.0}}"
```

**TF frames:** `map → odom → base_footprint → base_link → base_scan`
(`map → odom` only exists while SLAM or Nav2 is running.) Make a diagram with `ros2 run tf2_tools view_frames`.

---

## 5️⃣ SLAM — build a map

Keep **Terminal 1 (Gazebo)** running. Close RViz2 and teleop, then:

```bash
# Terminal 2
ros2 launch slam_toolbox online_async_launch.py use_sim_time:=true

# Terminal 3
ros2 launch turtlebot3_bringup rviz2.launch.py
#   RViz2: Fixed Frame = map, Add → By topic → /map → Map

# Terminal 4
ros2 run turtlebot3_teleop teleop_keyboard
```

Drive **slowly** around the world; the map fills in. Fast spinning makes a blurry map. Save it:

```bash
ros2 run nav2_map_server map_saver_cli -f ~/turtlebot3_map
```

This creates `~/turtlebot3_map.yaml` and `~/turtlebot3_map.pgm`.

## 6️⃣ Nav2 — autonomous navigation

Stop SLAM and teleop (Gazebo keeps running), then:

```bash
ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=True map:=$HOME/turtlebot3_map.yaml
```

In the RViz2 window that opens:

1. **2D Pose Estimate** → click and drag where the robot is, in the direction it faces.
2. **Nav2 Goal** → click and drag the destination.
3. Watch the robot plan a path and drive there. 🟢

---

## 🛠️ Troubleshooting

**Golden rule:** when anything strange happens, stop everything and restart cleanly:

```bash
killall -9 gzserver gzclient 2>/dev/null
ros2 daemon stop && ros2 daemon start
```

then start **only** Terminal 1 and run `bash check_simulation.sh`.

| Symptom | Cause | Fix |
| --- | --- | --- |
| Gazebo window is **black** (WSL2) | graphics driver | New terminal (`echo $LIBGL_ALWAYS_SOFTWARE` must print `1`); `wsl --update` + `wsl --shutdown` in PowerShell |
| Gazebo grey/stuck for minutes | first-run cache, or stale processes | Wait up to 3 min once; otherwise `killall -9 gzserver gzclient` and relaunch |
| `package 'turtlebot3_gazebo' not found` | terminal has no ROS settings | Open a **new** terminal (or `source ~/.bashrc`) |
| **Robot invisible / "small circle gets smaller"** | robot is **falling** – no floor | See below |
| Robot not listed in Gazebo (**World → Models → `burger`**) | spawn failed | See "Robot not spawned" below |
| Teleop numbers change but robot does not move | robot not spawned / falling / key presses too small | Run `check_simulation.sh`; press `w` 10–15 times |
| Robot visible in Gazebo, invisible in RViz2 | RViz2 zoomed out / Fixed Frame wrong | Section 3, RViz2 table |
| `bad interpreter: /bin/bash^M` | Windows line endings | `sed -i 's/\r$//' ~/tb3_sim/*.sh` |
| `rosdep` / apt errors | no internet / locked apt | check internet; `sudo apt update`; re-run the setup script |

### ❌ Robot falls through the floor (`z` very negative)

`check_simulation.sh` shows *"Robot is FALLING"*, or:

```bash
ros2 topic echo /odom --once --field pose.pose.position.z     # should be close to 0.0
```

shows something like `-881523`. Gazebo could not load `ground_plane` / `sun`.

**Fix:** re-run `bash setup_turtlebot3_sim.sh` (it recreates the floor if needed), open a **new terminal**, then:

```bash
echo $GAZEBO_MODEL_PATH            # should contain /usr/share/gazebo-11/models and ~/.gazebo/models
ls /usr/share/gazebo-11/models ~/.gazebo/models     # ground_plane and sun must appear in one of them
killall -9 gzserver gzclient 2>/dev/null
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

Gazebo should now show shadows and the robot should sit on the floor. In `~/.bashrc` the line `source /usr/share/gazebo/setup.sh` must come **before** `export GAZEBO_MODEL_DATABASE_URI=""`.

### ❌ Robot not spawned

In the Terminal 1 log look for `spawn_entity.py … process has finished cleanly`. Errors like these mean it failed:

```text
Service /spawn_entity unavailable. Was Gazebo started with GazeboRosFactory?
Entity [burger] already exists
```

Run the golden-rule clean restart. Launch **only one** simulation at a time (never `empty_world` and `turtlebot3_world` together) and wait for the log line before opening anything else. With software rendering the first spawn can take up to a minute.

---

## 📋 Cheat sheet

```bash
# Start (3 terminals)
killall -9 gzserver gzclient 2>/dev/null && ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
ros2 launch turtlebot3_bringup rviz2.launch.py
ros2 run turtlebot3_teleop teleop_keyboard

# Health check
bash ~/tb3_sim/check_simulation.sh

# Handy tools
ros2 node list          ros2 topic list         ros2 topic hz /scan
ros2 topic echo /odom --once                    ros2 run tf2_tools view_frames
```

---

## 🧑‍🏫 Notes for instructors

- Tested design: Ubuntu 22.04 + ROS 2 Humble + Gazebo Classic 11 (apt packages, no source build).
- The Humble TurtleBot3 simulation uses the Gazebo **entity name `burger`** (not `turtlebot3_burger`).
- Ask students to paste the output of `bash check_simulation.sh` when they need help — it shows exactly which stage fails.
- Common classroom failures: (1) black Gazebo on WSL2 → software rendering + `wsl --update`; (2) no floor → robot falls; (3) opening RViz2/teleop before Gazebo has finished spawning the robot; (4) leftover `gzserver` processes.

## 📚 Learning path

ROS 2 basics → Nodes & topics → TurtleBot3 → Gazebo → Teleoperation → LiDAR → TF2 → RViz2 → SLAM → Mapping → Nav2 → Autonomous navigation → Real robot

**Technologies:** ROS 2 Humble · Ubuntu 22.04 · WSL2 · Gazebo Classic · TurtleBot3 · RViz2 · SLAM Toolbox · Nav2 · TF2
