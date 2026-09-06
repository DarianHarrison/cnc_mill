# first_cnc_mill

![Alt Text](./trv.png)

# CNC Milling Standard Operating Procedure (SOP)

This guide documents the end-to-end workflow for designing, prepping, and safely executing CNC milling jobs on a Genmitsu 3018 CNC (or similar desktop routers), using Inkscape (Vector), Kiri:Moto (CAM), and CNCjs (Sender).

## 1. Hardware & Environment Profile
* **Machine:** Genmitsu 3018 (Max physical travel: 300mm X, 180mm Y)
* **Spindle:** 775 Brushed DC Motor
* **Control Software:** GRBL (via CNCjs)
* **Tooling Lifecycle:** Micro-tooling (e.g., 2mm carbide endmills) are consumables. Dull bits should be stored safely and sold to specialized scrap yards ($15-$23/lb for pure solid tungsten carbide).

---

## 2. Phase 1: Vector Design & Feasibility (Inkscape)

### The Stroke Simulation Trick
Before exporting, mathematically prove the bit fits inside the design geometries to prevent the CAM software from generating self-intersecting blobs.
1. Select the entire design (`Ctrl+A`).
2. Remove solid color fills (`No paint`).
3. Apply a solid **Stroke color**.
4. Set the **Stroke width** strictly 0.1mm larger than your endmill (e.g., `2.1mm` for a `2mm` bit).
5. **Verify:** Zoom in on tight corners and inner loops (like "e" or "a"). If the lines pinch completely shut, the bit is too large. Fix by changing the font, switching to a V-bit, or scaling the design up.

### Physical Boundary Constraints
* Leave a strict 10mm-20mm safety margin on all sides to prevent gantry crashes. 
* For a 300x180mm bed, scale the absolute maximum design bounding box to **280x160mm**.
* Once verified, convert all text/shapes to raw vectors (`Path > Object to Path`) before saving as SVG.

---

## 3. Phase 2: CAM Setup (Kiri:Moto)

Configure safe material removal rates. Default baseline for MDF:
* **Step Down:** `0.5mm` (Never force deep single passes).
* **Target Depth:** Ensure your total depth is set (e.g., `1.5mm`).
* **Z-Bottom Override:** Leave the `Z Bottom` parameter **blank** unless explicitly necessary, as filling it can cause CAM calculation errors or force the bit to skip the cut entirely.
* **Visual Verification:** Always preview the 3D toolpath. Manually count the layer lines (e.g., three visible layers for a 1.5mm total depth at a 0.5mm step-down). Export as `.nc` G-code.

---

## 4. Phase 3: Machine Setup & Zeroing (CNCjs)

### Strategic Clamping
Position all clamps on the extreme outer perimeter of the stock. Manually jog the spindle over the clamps to verify it will not collide during its maximum X/Y travel.

### Zeroing Coordinates
1. **X and Y:** Jog to the bottom-left corner of the stock. Zero both axes (Map Pin icons).
2. **Z Axis (Paper Trick):** Place printer paper under the bit. Drop the jog step to `0.1mm`. Lower the Z-axis until it slightly drags the paper. Zero the Z-axis.
3. **Safety Retract:** Immediately change the jog step to `10mm` and lift the Z-axis into the air. **Never start the spindle while touching the stock.**

---

## 5. Phase 4: Pre-Flight & Execution

### Final Software Checks
* **Z Min Verification:** Look at the G-code bounding box in CNCjs. Verify `Z Min` perfectly matches your CAM target depth (e.g., `-1.510 mm`).
* **Queue Verification:** Ensure the G-code is fully loaded (Lines Sent: `0 / [Total]`).

### State Management & Troubleshooting
If you hit "Play" and nothing happens, the GRBL board is likely paused:
* **Clear the Hold:** If an orange **Hold** badge is visible (often triggered by the Feedhold `!` command), click **Cycle Start** (`~` in top right) to return the machine to **Idle**.
* **Soft Reset:** If the connection is completely stale, click the red **Reset** button, click **Unlock**, re-upload the G-code file, and hit Play. (This preserves your zero coordinates).
* **Emergency Hard Kill:** If the bit plunges too deep or the machine crashes, physically yank the power cord. Do not rely on software buttons during a hardware collision.

---

## 6. Phase 5: Post-Job & Maintenance

### Static Safety & Dust Management
* **MDF Warning:** Never use a plastic vacuum wand near the spinning metal spindle during a carve. The fine dust generates static electricity that can arc to the frame, shorting the USB connection and ruining the job.
* **Cleanup:** Wait 1-2 minutes for toxic dust (MDF glue/resins) to settle. Jog the bit away, completely power off the machine, and *then* vacuum or sweep. Clear fuzzy edges on MDF with a stiff dry brush or Scotch-Brite pad.

### Maintenance Cycles (Every 10-20 Hours)
* **Lubrication:** Apply dry PTFE spray to X, Y, and Z threaded rods. Never use wet/heavy greases, which will turn MDF dust into concrete.
* **Hardware:** Check and tighten the grub screws on the motor couplers and the main frame bolts, which vibrate loose over time.

---

## 7. Future Material Adaptations

When migrating away from MDF, adjust tooling and CAM parameters:
* **Acrylic/Plastics:** Standard flat endmills will melt the plastic and snap. Switch to a **Single Flute Upcut** bit, increase the feed rate significantly to prevent heat buildup, and use shallow step-downs.
* **Hardwoods (Oak/Maple):** Much denser than MDF. Reduce feed rates, reduce step-down depth, and use "ramping" (plunging at an angle rather than straight down) to prevent burning the wood.
