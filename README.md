**Please Note:** I'm using GameMode here instead of GameInstance so the team can see how the item array behaves—specifically (i exposed it so its instance editable), how moving items around in the inventory also affects their position in persistent data. In the actual game, I'll be using GameInstance exclusively.


# FlawlessAbbeyTestAssignment
## How to Play-test

1. **Launch the project** and load the test scene. You will see three interactable objects:

   * NPC
   * 2D Jug
   * 3D Box

2. ### NPC Interaction

   * Approach the NPC and press **E** to start the dialogue loop.
   * Two lines of dialogue cycle, then reset.
   * Talk to the NPC from unusual angles and camera orientations to expose edge cases.

3. ### 3D Box Workflow

   * Pick up the box to add it to your inventory.
   * Press **I** to open (or **I** or **Esc** to close) the inventory.
   * Drag the box from its slot into the preview pane beneath the four inventory slots.
   * You can rotate and manipulate the box in the preview pane.
   * Dragging the slot containing the box onto an empty slot moves the box there and clears its original slot.

4. ### 2D Jug Workflow

   * Repeat the steps above with the jug.
   * After placing the jug in the preview pane, click **Forget Note**.
   * The jug should vanish from the preview pane, the inventory, and the underlying Game Mode data while leaving other items intact.

## Dialogue System

### **Interface**

* All interactables implement BPI_Interactable with three methods: Interact, ShowNextLine, and FinishInteraction.
* This keeps the API uniform and avoids casting.

### **Parent Blueprints**

* Three parent blueprints hold shared logic for NPCs and world props.
* Key variables (CurrentRow, ResetRow) are exposed so level designers can tweak them in the editor.

### **Data-Driven Dialogue**

* A struct stores CurrentRowID and NextRowID.
* Dialogue text lives in a data table, making localization and edits simple.

### **Interaction Flow**

1. Interact locks player input and shows the current line.
2. A timer reveals text at TextRevealSpeed.
3. ShowNextLine loads the next row; if none exists, FinishInteraction unlocks movement and resets the dialogue.

### **HUD Integration**

* One HUD widget contains both the inventory and a speech-bubble widget, handling show/hide animations and text updates.

### **Why This Approach**

* Interfaces remove costly casts and keep player code generic.
* Parent blueprints stop logic duplication.
* Data tables let non-programmers update dialogue without touching code, speeding up iteration.


## Inventory, Drag-and-Drop, and Viewport

### Data Flow

* **GameInstance** stores a persistent Items array (struct: name, amount, icon, mesh).
* **GameMode** copies this array on level load and **creates the HUD**, which contains the Inventory and Viewport widgets.
* **WBP_Inventory** runs LoadInventory, loops through Items, spawns SlotWidgets in a WrapBox, and removes stale slots before rebuilding the grid.

### Item Definitions

* Each pickup is a blueprint with one exposed variable: ItemName.
* On BeginPlay the blueprint looks up`ItemName in an item Data Table, fills the struct, and pushes it to GameMode (which then syncs GameInstance).

### Drag-and-Drop

* SlotWidgets override UE drag events, show a DragWidget, and pass item data in the payload.
* Dropping on another slot swaps items; dropping on the preview pane triggers the viewport logic.

### Viewport Preview

* **2D item**: widget enlarges the icon.
* **3D item**: pane leverages a spawned BP_Rotate3DItem, assigns the mesh, and feeds a render target back to the widget. A simple tick rotates the actor so users can spin the model or zoom in.

### Forget Note

* The Forget button replaces the note’s slot with an empty struct, then I remove the entry from GameMode Items array, and I update GameInstance, which refreshes the UI.

### Casting Notes

* The system casts to **GameMode** (to reach the HUD and items array).
* This cast is safe and inexpensive because GameMode is spawned once per level and never destroyed, eliminating dangling-reference risks.

### Why This Approach

* **Persistence:** GameInstance keeps items across level loads.
* **Editor-friendly:** designers only enter an item name; the data table supplies the rest.
* **Performance:** casting is limited to stable framework classes; drag payloads stay lightweight; the 3D preview renders once and reuses its target.
