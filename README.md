# Report: Distributed Shape Detection – Circle, Cross, and H Patterns
**Authors:** Kyllian Cuevas, Pierre Meyer, Thomas Mirbey  
**Program:** M2 IoT 

---

## 1. Overview

This code is our implementation of the **distributed shape recognition system** for blinky blocks. The algorithm allows the network of modules to **automatically detect geometric shapes** such as a **circle**, **cross (+)**, and **H** formation. 

Each block behaves autonomously and communicates only with directly connected neighbors. Color variations on the LEDs represent detection states as described in the README. 

---

## 2. Code Structure

The architecture is divided into **three main parts**:

1. **Initialization (`BBinit`)**  
   - Detects all active neighbors.  
   - Generates a random ID (`my_id`) to differentiate messages.  
   - Resets all detection states via `circle_reset_state()`. 

2. **Main Loop (`BBloop`)**  
   - Periodically polls for connection changes.  
   - Manages shape detection priorities (Circle → Cross → H).  
   - Periodically triggers leader election and ring tests.

3. **Packet Processing (`process_standard_packet`)**  
   - Handles all inter-node messages like `ADD`, `REMOVE`, `PLUS`, `H`, `HR`, `HRGO`, `MSG_RING_STEP`, and `MSG_RING_COLOR`.  
   - Uses message IDs and conditional forwarding to prevent repeated message processing.  
   - Performs LED updates according to shape detection progress. 

---

## 3. Avoiding Loops

To prevent infinite message loops or redundant propagation, the mechanism employs several safeguards:

- **Unique Message IDs**:  
  Each transmitted message embeds a randomized identifier `msg_id` computed using `(id_msg ^ my_id) & 0xFF`.  
  When a block receives a message, it compares `msg_id` with `last_msg_id`.  
  - If identical → the message is ignored.  
  - This prevents circular forwarding among already-visited nodes.

- **Directional Forwarding**:  
  For events like `RESET` or `ADD/REMOVE`, packets are re-sent to all connected ports **except the incoming one**.  
  This stops back-propagation loops.

- **Ring and Color Step Tracking**:  
  The circle sequence uses `ring_step_seen` and `color_step_seen` to store the last step processed.  
  Any repeated token (from stale or delayed paths) is disregarded, maintaining a clean traversal path.

---

## 4. Shape Testing Procedure

To validate the algorithm, three physical or simulated structures can be tested:

1. **Cross (“+”) Shape:**  
   - Connect one module to four neighbors (N, S, E, W).  
   - Observe a **cyan** center and **blue** arms. 

2. **H Shape:**  
   - Create a vertical line of three blocks.  
   - Add side neighbors to the top and bottom blocks.  
   - The center becomes **purple**, and successful arms stay **purple** if validation passes. 

3. **Circle (Ring) Shape:**  
   - Arrange eight blocks in a loop following the sequence  
     `WEST → SOUTH → SOUTH → EAST → EAST → NORTH → NORTH → WEST`.  
   - The elected leader (white) initiates ring tests, followed by **red** coloring when the loop closes successfully. 

In each case, removing or adding a block triggers a global **RESET**, clearing previous detections and ensuring fresh evaluation.

---

## 5. LED Colors During Circle Detection

This guide explains **all LED colors** you may see during the circle detection process, the **cross (+)** pattern and the **H** pattern in the order they appear.

### Start / Reset → **Black**
- All blocks turn **black** (off).  
- The system resets all states and starts evaluating the structure again.

### Early H attempts → **Purple flashes**
- A potential H-center turns **purple**.  
- If the H conditions are not satisfied, the attempt **fails** and returns to **black**. 

### Circle leader detected → **White**
- One block becomes the **leader** and turns **white**.  
- It sends a "ring test" token around the loop. 

### Circle validated → **White (leader)**
- The ring is confirmed to be valid.  
- Leader stays **white** before starting coloring phase. 

### Circle coloring → **Red**
- Every block on the ring becomes **red**.  
- Final result: **All ring blocks are red**. 

### Any structural change → **Black**
- A RESET is broadcast and all LEDs return to **black**. 

---

## 6. Additional Demo and Resources

A video demonstration illustrates the shape transitions and LED states:  
[![Demo Video](https://www.canva.com/design/DAG6j2xt-qs/kHveo4VeeYsWkw4HTbUqwQ/watch?utm_content=DAG6j2xt-qs&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h2976927901)](https://www.canva.com/design/DAG6j2xt-qs/kHveo4VeeYsWkw4HTbUqwQ/watch) 





