# Report: Distributed Shape Detection – Circle, Cross, and H Patterns
**Authors:** Kyllian Cuevas, Pierre Meyer, Yohann Mitel, Thomas Mirbey  
**Program:** M2 IoT [web:1]

---

## 1. Overview

This project implements a **distributed topology recognition system** for modular units (BBs), each equipped with ports (North, South, East, West), LEDs, and serial communication. The algorithm allows the network of modules to **automatically detect geometric shapes** such as a **circle**, **cross (+)**, and **H** formation. [web:2]

Each block behaves autonomously and communicates only with directly connected neighbors. Color variations on the LEDs represent detection states as described in the README. [web:1]

---

## 2. Code Structure

The architecture is divided into **three main parts**:

1. **Initialization (`BBinit`)**  
   - Detects all active neighbors.  
   - Generates a random ID (`my_id`) to differentiate messages.  
   - Resets all detection states via `circle_reset_state()`. [web:1]

2. **Main Loop (`BBloop`)**  
   - Periodically polls for connection changes.  
   - Manages shape detection priorities (Circle → Cross → H).  
   - Periodically triggers leader election and ring tests. [web:3]

3. **Packet Processing (`process_standard_packet`)**  
   - Handles all inter-node messages like `ADD`, `REMOVE`, `PLUS`, `H`, `HR`, `HRGO`, `MSG_RING_STEP`, and `MSG_RING_COLOR`.  
   - Uses message IDs and conditional forwarding to prevent repeated message processing.  
   - Performs LED updates according to shape detection progress. [web:1]

The modular design ensures that topology logic and communication feedback remain isolated and reusable. [web:3]

---

## 3. Avoiding Loops

To prevent infinite message loops or redundant propagation, the mechanism employs several safeguards:

- **Unique Message IDs**:  
  Each transmitted message embeds a randomized identifier `msg_id` computed using `(id_msg ^ my_id) & 0xFF`.  
  When a block receives a message, it compares `msg_id` with `last_msg_id`.  
  - If identical → the message is ignored.  
  - This prevents circular forwarding among already-visited nodes. [web:1]

- **Directional Forwarding**:  
  For events like `RESET` or `ADD/REMOVE`, packets are re-sent to all connected ports **except the incoming one**.  
  This stops back-propagation loops. [web:3]

- **Ring and Color Step Tracking**:  
  The circle sequence uses `ring_step_seen` and `color_step_seen` to store the last step processed.  
  Any repeated token (from stale or delayed paths) is disregarded, maintaining a clean traversal path. [web:1]

Together, these constraints ensure that all broadcasts, tokens, and propagation waves terminate gracefully without feedback or message storms. [web:2]

---

## 4. Shape Testing Procedure

To validate the algorithm, three physical or simulated structures can be tested:

1. **Cross (“+”) Shape:**  
   - Connect one module to four neighbors (N, S, E, W).  
   - Observe a **cyan** center and **blue** arms. [web:1]

2. **H Shape:**  
   - Create a vertical line of three blocks.  
   - Add side neighbors to the top and bottom blocks.  
   - The center becomes **purple**, and successful arms stay **purple** if validation passes. [web:2]

3. **Circle (Ring) Shape:**  
   - Arrange eight blocks in a loop following the sequence  
     `WEST → SOUTH → SOUTH → EAST → EAST → NORTH → NORTH → WEST`.  
   - The elected leader (white) initiates ring tests, followed by **red** coloring when the loop closes successfully. [web:1]

In each case, removing or adding a block triggers a global **RESET**, clearing previous detections and ensuring fresh evaluation. [web:3]

---

## 5. LED Colors During Circle Detection

This guide explains **all LED colors** you may see during the circle detection process, the **cross (+)** pattern and the **H** pattern in the order they appear.

### Start / Reset → **Black**
- All blocks turn **black** (off).  
- The system resets all states and starts evaluating the structure again. [web:1]

### Early H attempts → **Purple flashes**
- A potential H-center turns **purple**.  
- If the H conditions are not satisfied, the attempt **fails** and returns to **black**. [web:2]

### Circle leader detected → **White**
- One block becomes the **leader** and turns **white**.  
- It sends a "ring test" token around the loop. [web:1]

### Circle validated → **White (leader)**
- The ring is confirmed to be valid.  
- Leader stays **white** before starting coloring phase. [web:3]

### Circle coloring → **Red**
- Every block on the ring becomes **red**.  
- Final result: **All ring blocks are red**. [web:1]

### Any structural change → **Black**
- A RESET is broadcast and all LEDs return to **black**. [web:2]

---

## 6. Additional Demo and Resources

A video demonstration illustrates the shape transitions and LED states:  
[![Demo Video](https://www.canva.com/design/DAG6j2xt-qs/kHveo4VeeYsWkw4HTbUqwQ/watch?utm_content=DAG6j2xt-qs&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h2976927901)](https://www.canva.com/design/DAG6j2xt-qs/kHveo4VeeYsWkw4HTbUqwQ/watch) [web:1]
