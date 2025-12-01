<script lang="ts">
    // --- Board state och utility-funktioner ---
    function resetBoard() {
        if (
            confirm(
                "Are you sure you want to reset the board? All lanes and tasks will be deleted."
            )
        ) {
            localStorage.removeItem(STORAGE_KEY);
            location.reload();
        }
    }
    // --- UI state ---
    let showSettings = false;
    // --- Komponenter ---
    import Swimlane from "./Swimlane.svelte";
    // --- Typer ---
    type Task = { id: number; title: string };
    type Lane = { id: number; title: string; tasks: Task[] };

    // --- Props ---
    export let lanes: Lane[] = [];

    // --- LocalStorage key ---
    const STORAGE_KEY = "projecto-board";

    // --- Läs board från localStorage om det finns ---
    function loadBoard(): Lane[] {
        try {
            const raw = localStorage.getItem(STORAGE_KEY);
            if (raw) {
                return JSON.parse(raw);
            }
        } catch {}
        return lanes.map((lane) => ({ ...lane, tasks: [...lane.tasks] }));
    }

    let boardLanes: Lane[] = loadBoard();

    // --- Spara board till localStorage ---
    function saveBoard() {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(boardLanes));
    }

    // --- Hantering av lanes ---
    let newLaneTitle = "";

    function addLane() {
        if (newLaneTitle.trim()) {
            const newId = Math.max(0, ...boardLanes.map((l) => l.id)) + 1;
            boardLanes = [
                ...boardLanes,
                { id: newId, title: newLaneTitle, tasks: [] },
            ];
            newLaneTitle = "";
            saveBoard();
        }
    }

    function removeLane(laneId: number) {
        boardLanes = boardLanes.filter((l) => l.id !== laneId);
        saveBoard();
    }

    function addTask(laneId: number, title: string) {
        const lane = boardLanes.find((l: Lane) => l.id === laneId);
        if (lane) {
            const newId =
                Math.max(
                    0,
                    ...boardLanes.flatMap((l: Lane) =>
                        l.tasks.map((t: Task) => t.id)
                    )
                ) + 1;
            lane.tasks.push({ id: newId, title });
            boardLanes = boardLanes.map((l) => ({ ...l, tasks: [...l.tasks] }));
            saveBoard();
        }
    }

    // --- Lane edit modal state ---
    let editingLane: Lane | null = null;
    let editingLaneTitle = "";
    function openEditLane(lane: Lane) {
        editingLane = lane;
        editingLaneTitle = lane.title;
    }
    function closeEditLane() {
        editingLane = null;
        editingLaneTitle = "";
    }
    function saveEditLane() {
        if (editingLane !== null && editingLaneTitle.trim()) {
            const laneId = editingLane.id;
            boardLanes = boardLanes.map((l) =>
                l.id === laneId ? { ...l, title: editingLaneTitle.trim() } : l
            );
            saveBoard();
            closeEditLane();
        }
    }

    // --- Flytta task mellan lanes ---
    function handleMoveTask(
        e: CustomEvent<{ taskId: number; fromLaneId: number; toLaneId: number }>
    ) {
        const { taskId, fromLaneId, toLaneId } = e.detail;
        if (fromLaneId === toLaneId) return;
        const fromLane = boardLanes.find((l) => l.id === fromLaneId);
        const toLane = boardLanes.find((l) => l.id === toLaneId);
        if (!fromLane || !toLane) return;
        const taskIdx = fromLane.tasks.findIndex((t) => t.id === taskId);
        if (taskIdx === -1) return;
        const [task] = fromLane.tasks.splice(taskIdx, 1);
        toLane.tasks.push(task);
        // Trigger reactivity
        boardLanes = boardLanes.map((l) => ({ ...l, tasks: [...l.tasks] }));
        saveBoard();
    }

    // --- Drag & drop för lanes ---
    let draggedLaneId: number | null = null;

    function handleLaneDragStart(laneId: number) {
        draggedLaneId = laneId;
    }

    function handleLaneDragOver(event: DragEvent) {
        event.preventDefault();
    }

    function handleLaneDrop(targetLaneId: number) {
        if (draggedLaneId === null || draggedLaneId === targetLaneId) return;
        const fromIndex = boardLanes.findIndex((l) => l.id === draggedLaneId);
        const toIndex = boardLanes.findIndex((l) => l.id === targetLaneId);
        if (fromIndex === -1 || toIndex === -1) return;
        const updated = [...boardLanes];
        const [moved] = updated.splice(fromIndex, 1);
        updated.splice(toIndex, 0, moved);
        boardLanes = updated;
        draggedLaneId = null;
        saveBoard();
    }

    // --- Add task modal state och handler ---
    let showAddTask = false;
    let newTaskTitle = "";
    let newTaskLaneId: number | null = null;

    function handleAddTask() {
        if (newTaskTitle.trim() && newTaskLaneId !== null) {
            addTask(newTaskLaneId, newTaskTitle.trim());
            newTaskTitle = "";
            newTaskLaneId = null;
            showAddTask = false;
        }
    }

    // --- Task details modal state ---
    let showTaskDetails = false;
    let selectedTask: Task | null = null;
    let selectedLaneId: number | null = null;
    let editingTaskTitle = "";

    function openTaskDetails(task: Task, laneId: number) {
        selectedTask = task;
        selectedLaneId = laneId;
        editingTaskTitle = task.title;
        showTaskDetails = true;
    }
    function closeTaskDetails() {
        showTaskDetails = false;
        selectedTask = null;
        selectedLaneId = null;
        editingTaskTitle = "";
    }
    function saveTaskEdit() {
        if (
            selectedTask &&
            selectedLaneId !== null &&
            editingTaskTitle.trim()
        ) {
            const lane = boardLanes.find((l) => l.id === selectedLaneId);
            if (lane) {
                lane.tasks = lane.tasks.map((t) =>
                    t.id === selectedTask!.id
                        ? { ...t, title: editingTaskTitle.trim() }
                        : t
                );
                boardLanes = boardLanes.map((l) => ({
                    ...l,
                    tasks: [...l.tasks],
                }));
                saveBoard();
                closeTaskDetails();
            }
        }
    }
    function removeTask(laneId: number, taskId: number) {
        const lane = boardLanes.find((l) => l.id === laneId);
        if (lane) {
            lane.tasks = lane.tasks.filter((t) => t.id !== taskId);
            boardLanes = boardLanes.map((l) => ({ ...l, tasks: [...l.tasks] }));
            saveBoard();
            closeTaskDetails();
        }
    }
</script>

{#if showAddTask}
    <div
        class="modal-overlay"
        role="dialog"
        aria-modal="true"
        tabindex="0"
        on:click={() => (showAddTask = false)}
        on:keydown={(e) => e.key === "Escape" && (showAddTask = false)}
    >
        <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div class="modal" role="document" on:click|stopPropagation>
            <h3>Skapa ny task</h3>
            <form
                on:submit|preventDefault={handleAddTask}
                style="display: flex; flex-direction: column; gap: 1rem;"
            >
                <input
                    type="text"
                    bind:value={newTaskTitle}
                    placeholder="Task titel..."
                    required
                />
                <select bind:value={newTaskLaneId} required>
                    <option value="" disabled selected>Välj lane</option>
                    {#each boardLanes as lane}
                        <option value={lane.id}>{lane.title}</option>
                    {/each}
                </select>
                <button type="submit">Skapa</button>
            </form>
            <div class="modal-actions">
                <button type="button" on:click={() => (showAddTask = false)}
                    >Stäng</button
                >
            </div>
            <button
                class="reset-board-btn modal-margin-left"
                type="button"
                aria-label="Reset board"
                on:click={resetBoard}
            >
                <span
                    class="material-icons"
                    style="font-size: 20px; color: #d32f2f; vertical-align: middle;"
                    >delete_forever</span
                >
                Reset Board
            </button>
        </div>
    </div>
{/if}

{#if showSettings}
    <div
        class="modal-overlay"
        role="dialog"
        aria-modal="true"
        tabindex="0"
        on:click={() => (showSettings = false)}
        on:keydown={(e) => e.key === "Escape" && (showSettings = false)}
    >
        <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div class="modal" role="document" on:click|stopPropagation>
            <h3>Inställningar</h3>
            <form class="add-lane-form" on:submit|preventDefault={addLane}>
                <input
                    type="text"
                    bind:value={newLaneTitle}
                    placeholder="Ny swimlane..."
                />
                <button type="submit">Lägg till lane</button>
            </form>
            <div
                style="margin-top: 2rem; display: flex; flex-direction: column; gap: 0.5rem;"
            >
                <button
                    type="button"
                    class="reset-board-btn modal-danger-btn"
                    on:click={resetBoard}
                >
                    <span class="material-icons danger-icon"
                        >delete_forever</span
                    >
                    Återställ board
                </button>
                <button type="button" on:click={() => (showSettings = false)}>
                    Stäng
                </button>
            </div>
        </div>
    </div>
{/if}

<div class="board">
    <div class="board-actions">
        <button
            class="add-task-btn"
            type="button"
            aria-label="Skapa ny task"
            on:click={() => (showAddTask = true)}
        >
            <svg
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
            >
                <circle
                    cx="12"
                    cy="12"
                    r="10"
                    stroke="#1976d2"
                    stroke-width="2"
                    fill="none"
                />
                <path
                    d="M12 8V16M8 12H16"
                    stroke="#1976d2"
                    stroke-width="2"
                    stroke-linecap="round"
                />
            </svg>
        </button>
        <button
            class="settings-btn"
            type="button"
            aria-label="Inställningar"
            on:click={() => (showSettings = true)}
        >
            <span class="material-icons" style="font-size: 24px; color: #555;"
                >settings</span
            >
        </button>
    </div>
    <div class="board-inner">
        {#each boardLanes as lane (lane.id)}
            <div
                class="lane-wrapper"
                role="listitem"
                on:dragover|preventDefault={handleLaneDragOver}
                on:drop={() => handleLaneDrop(lane.id)}
            >
                <div
                    class="lane-header"
                    role="button"
                    tabindex="0"
                    draggable="true"
                    on:dragstart={() => handleLaneDragStart(lane.id)}
                >
                    <span class="lane-title">{lane.title}</span>
                    <button
                        class="edit-lane-btn"
                        type="button"
                        aria-label="Redigera lane"
                        on:click={() => openEditLane(lane)}
                    >
                        <span
                            class="material-icons"
                            style="font-size: 20px; color: #555;">settings</span
                        >
                    </button>
                </div>
                <Swimlane
                    {lane}
                    on:addTask={(e) => addTask(lane.id, e.detail.title)}
                    on:moveTask={handleMoveTask}
                    on:taskClick={(e) =>
                        openTaskDetails(e.detail.task, lane.id)}
                />
                {#if showTaskDetails && selectedTask}
                    <div
                        class="modal-overlay"
                        role="dialog"
                        aria-modal="true"
                        tabindex="0"
                        on:click={closeTaskDetails}
                        on:keydown={(e) =>
                            e.key === "Escape" && closeTaskDetails()}
                    >
                        <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
                        <!-- svelte-ignore a11y-click-events-have-key-events -->
                        <div
                            class="modal"
                            role="document"
                            on:click|stopPropagation
                            on:keydown={(e) =>
                                e.key === "Enter" && e.stopPropagation()}
                        >
                            <h3>Edit Task</h3>
                            <form
                                on:submit|preventDefault={saveTaskEdit}
                                style="margin-bottom: 1rem;"
                            >
                                <label for="edit-task-title"
                                    ><strong>Title:</strong></label
                                >
                                <input
                                    id="edit-task-title"
                                    type="text"
                                    bind:value={editingTaskTitle}
                                    required
                                />
                            </form>
                            <div class="modal-actions">
                                <button type="button" on:click={saveTaskEdit}
                                    >Save</button
                                >
                                <button
                                    type="button"
                                    on:click={closeTaskDetails}>Close</button
                                >
                                {#if selectedLaneId !== null && selectedTask}
                                    <button
                                        class="remove-task"
                                        type="button"
                                        on:click={() =>
                                            removeTask(
                                                selectedLaneId!,
                                                selectedTask!.id
                                            )}>Delete</button
                                    >
                                {/if}
                            </div>
                        </div>
                    </div>
                {/if}
            </div>
        {/each}

        {#if editingLane}
            <div
                class="modal-overlay"
                role="dialog"
                aria-modal="true"
                tabindex="0"
                on:click={closeEditLane}
                on:keydown={(e) => e.key === "Escape" && closeEditLane()}
            >
                <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <div class="modal" role="document" on:click|stopPropagation>
                    <h3>Redigera lane</h3>
                    <input
                        type="text"
                        bind:value={editingLaneTitle}
                        aria-label="Lane titel"
                    />
                    <div class="modal-actions">
                        <button type="button" on:click={saveEditLane}
                            >Spara</button
                        >
                        <button type="button" on:click={closeEditLane}
                            >Avbryt</button
                        >
                        <button
                            type="button"
                            class="remove-lane"
                            on:click={() => {
                                if (editingLane !== null) {
                                    removeLane(editingLane.id);
                                    closeEditLane();
                                }
                            }}>Ta bort lane</button
                        >
                    </div>
                </div>
            </div>
        {/if}
    </div>
</div>

<style>
    @import url("https://fonts.googleapis.com/icon?family=Material+Icons");
    :global(body) {
        font-family: Calibri, Arial, Helvetica, sans-serif;
    }
    .add-task-btn {
        background: none;
        border: none;
        cursor: pointer;
        padding: 0.2rem;
        display: flex;
        align-items: center;
    }

    .board {
        display: flex;
        flex: 1;
        min-height: 0;
        background: #f4f4f4;
        position: relative;
        overflow-y: auto;
        min-height: 0;
    }
    .board-actions {
        position: fixed;
        top: 0.5rem;
        right: 1rem;
        display: flex;
        gap: 0.5rem;
        z-index: 100;
    }
    .board-inner {
        width: 100%;
        padding: 0;
        box-sizing: border-box;
        display: flex;
        gap: 1rem;
    }

    .settings-btn {
        background: none;
        border: none;
        cursor: pointer;
        padding: 0.2rem;
        display: flex;
        align-items: center;
    }

    .add-lane-form {
        display: flex;
        gap: 0.5rem;
        margin-bottom: 1rem;
    }
    .add-lane-form input {
        flex: 1;
        padding: 0.3rem 0.5rem;
        border-radius: 4px;
        border: 1px solid #ccc;
    }
    .add-lane-form button {
        padding: 0.3rem 0.8rem;
        border-radius: 4px;
        border: none;
        background: #1976d2;
        color: #fff;
        cursor: pointer;
    }
    .lane-wrapper {
        display: flex;
        flex-direction: column;
        align-items: stretch;
        position: relative;
    }
    .lane-header {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        margin-bottom: 0.5rem;
    }
    .lane-title {
        font-size: 1.1rem;
        font-weight: bold;
        flex: 1;
    }
    .edit-lane-btn {
        background: none;
        border: none;
        cursor: pointer;
        padding: 0.1rem;
        display: flex;
        align-items: center;
    }
    .modal-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background: rgba(0, 0, 0, 0.3);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 1000;
    }
    .modal {
        background: #fff;
        padding: 2rem;
        border-radius: 8px;
        box-shadow: 0 2px 16px rgba(0, 0, 0, 0.15);
        min-width: 300px;
    }
    .modal-actions {
        display: flex;
        gap: 1rem;
        margin-top: 1rem;
        justify-content: flex-end;
    }
</style>
