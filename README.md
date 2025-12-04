# LED Colors During Circle Detection

This guide explains **all LED colors** you may see during the circle detection process, the **cross (+)** pattern and the **H** pattern in the order they appear in the video.

---

## 1. Start / Reset → **Black**

At startup or after a topology change (ADD/REMOVE + RESET):

- All blocks turn **black** (off).
- The system resets all states and starts evaluating the structure again.

---

## 2. Early H attempts → **Purple flashes**

Before the circle algorithm takes priority, the system may briefly try to detect an **H shape**.

- A potential H-center turns **purple**.
- If the H conditions are not satisfied (missing neighbors), the attempt **fails**:
  - The center goes back to **black**.
  - The surrounding blocks also return to **black**.

This is why you see **short purple flashes** before the circle logic takes over.  
These H attempts are harmless and always end quickly.

---

## 3. Circle leader detected → **White**

If the structure matches the conditions for a circle:

- One block becomes the **leader**.
- It turns **white**.
- It then sends a “ring test” token around the loop:

`WEST → SOUTH → SOUTH → EAST → EAST → NORTH → NORTH → WEST`

Other blocks remain **black** during this test.

---

## 4. Circle validated → **White (leader)**

If the token successfully returns to the leader:

- The ring is confirmed to be valid.
- The leader stays **white** for a moment before starting the coloring phase.

---

## 5. Circle coloring → **Red**

The leader sends a second token to color the circle:

- Every block on the ring becomes **red** when it receives `MSG_RING_COLOR`.
- The red color travels around the circle in the same direction.

Final result:

- **All ring blocks are red.**
- The leader also becomes **red** at the end.
- Blocks not on the ring stay **black**.

---

## 6. Any structural change → **Black**

If a block is added or removed:

- A RESET is broadcast.
- The circle, H, and cross states are cleared.
- All LEDs return to **black**, and the detection restarts.


# LED Colors During Cross (+) and H Detection

# 1. Cross (+) Detection

The cross is detected when a block has **4 neighbors** (N, S, E, W).

## Step 1 — Candidate center detected → **Cyan**
When a block detects 4 connected neighbors:

- It becomes the **cross center**.
- Its LED turns **cyan**.
- It broadcasts a `PLUS` message to notify its neighbors.

## Step 2 — Cross propagation → **Blue**
When neighbors receive the `PLUS` message:

- They turn **blue**, marking they belong to the cross arms.
- They spread the `O` (clear) message to remove any leftover H colors nearby.

## Final state
- The center stays **cyan**.
- The arms remain **blue**.
- Blocks outside the cross stay **black**.

---

# 2. H Detection

The H shape involves a **vertical bar** (N–center–S) and **horizontal connections** on the top and bottom blocks.

## Step 1 — H candidate → **Purple**
A block becomes a potential H-center when it has **both a north and a south neighbor**:

- It turns **purple**.
- It sends `H` messages to its north and south neighbors.

## Step 2 — Verification 
The north and south neighbors become **red** and check whether they each have **both left and right neighbors**:

- If yes → They return a **positive HR** (the H is possible).
- If no → They return a **negative HR** (the H fails).

## Step 3 — H success
If **both HR messages are positive**, the H is valid:

- The center stay **puple**.
- It sends `HRGO` to color the shape.
- The top and bottom blocks propagate the arms:
  - Vertical blocks and side blocks become **purple** again (H arms).

Final look:
- Center in **purple**.
- H arms in **purple**.

## Step 4 — H failure → **Black**
If either neighbor cannot form its horizontal connection:

- The center returns to **black**.
- Neighbors are cleared using `O` messages.

---

# 3. Interaction With Circle Detection
Circle detection has **priority** over both cross and H.

- If a circle is confirmed, cross and H processing are ignored.
- This avoids unwanted colors interfering with the ring.

---

# 4. Reset Behavior
Whenever a block is added or removed:

- A global `RESET` is sent.
- All cross and H colors are cleared.
- All blocks return to **black**.
- The detection process restarts automatically.

---

# 5. Important Note About Yellow Blocks

If you **remove a block** and then **place it back**, it may remain **yellow**.

**Reason:**  
Auto-boot is **not enabled** on that block, so it does **not restart its program automatically**, and it simply keep the color **yellow**.

You must reboot the whole blocks manually if you want it to rejoin the algorithm.

