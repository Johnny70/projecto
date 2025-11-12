<script lang="ts">
    let showSettings = false;
    import Swimlane from "./Swimlane.svelte";
    type Task = { id: number; title: string };
    type Lane = { id: number; title: string; tasks: Task[] };

    export let lanes: Lane[] = [];

    // LocalStorage key
    const STORAGE_KEY = "projecto-board";

    // Read from localStorage if available
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

    // Save to localStorage
    function saveBoard() {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(boardLanes));
    }

    // For creating a new lane
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

    // Edit lane modal state
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

    // Move task between lanes
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

    // Add task modal state and handler
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

    // Task details modal state
    let showTaskDetails = false;
    let selectedTask: Task | null = null;
    let selectedLaneId: number | null = null;

    function openTaskDetails(task: Task, laneId: number) {
        selectedTask = task;
        selectedLaneId = laneId;
        showTaskDetails = true;
    }
    function closeTaskDetails() {
        showTaskDetails = false;
        selectedTask = null;
        selectedLaneId = null;
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

<div class="board-toolbar">
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
                stroke="#555"
                stroke-width="2"
                fill="none"
            />
            <path
                d="M12 8V12L14 14"
                stroke="#555"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
            />
        </svg>
    </button>
</div>

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
        <div
            class="modal"
            role="document"
            on:click|stopPropagation
            on:keydown={(e) => e.key === "Enter" && e.stopPropagation()}
        >
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
        <div class="modal" role="document">
            <h3>Inställningar</h3>
            <form class="add-lane-form" on:submit|preventDefault={addLane}>
                <input
                    type="text"
                    bind:value={newLaneTitle}
                    placeholder="Ny swimlane..."
                />
                <button type="submit">Lägg till lane</button>
            </form>
            <div class="modal-actions">
                <button type="button" on:click={() => (showSettings = false)}
                    >Stäng</button
                >
            </div>
        </div>
    </div>
{/if}

<div class="board">
    {#each boardLanes as lane (lane.id)}
        <div class="lane-wrapper">
            <div class="lane-header">
                <span class="lane-title">{lane.title}</span>
                <button
                    class="edit-lane-btn"
                    type="button"
                    aria-label="Redigera lane"
                    on:click={() => openEditLane(lane)}
                >
                    <svg
                        width="20"
                        height="20"
                        viewBox="0 0 20 20"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                    >
                        <circle
                            cx="10"
                            cy="10"
                            r="8"
                            stroke="#555"
                            stroke-width="1.5"
                            fill="none"
                        />
                        <path
                            d="M10 6V10L12 12"
                            stroke="#555"
                            stroke-width="1.5"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                    </svg>
                </button>
            </div>
            <Swimlane
                {lane}
                on:addTask={(e) => addTask(lane.id, e.detail.title)}
                on:moveTask={handleMoveTask}
                on:taskClick={(e) => openTaskDetails(e.detail.task, lane.id)}
            />
            {#if showTaskDetails && selectedTask}
                <div
                    class="modal-overlay"
                    role="dialog"
                    aria-modal="true"
                    tabindex="0"
                    on:click={closeTaskDetails}
                    on:keydown={(e) => e.key === "Escape" && closeTaskDetails()}
                >
                    <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
                    <div
                        class="modal"
                        role="document"
                        on:click|stopPropagation
                        on:keydown={(e) =>
                            e.key === "Enter" && e.stopPropagation()}
                    >
                        <h3>Task detaljer</h3>
                        <div style="margin-bottom: 1rem;">
                            <strong>Titel:</strong>
                            {selectedTask.title}
                        </div>
                        <div class="modal-actions">
                            <button type="button" on:click={closeTaskDetails}
                                >Stäng</button
                            >
                            {#if selectedLaneId !== null && selectedTask}
                                <button
                                    type="button"
                                    class="remove-task"
                                    on:click={() =>
                                        removeTask(
                                            selectedLaneId!,
                                            selectedTask!.id
                                        )}>Ta bort</button
                                >
                            {/if}
                        </div>
                    </div>
                </div>
            {/if}
            <!-- Remove button is moved to the settings modal -->
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
            <div class="modal" role="document">
                <h3>Redigera lane</h3>
                <input
                    type="text"
                    bind:value={editingLaneTitle}
                    aria-label="Lane titel"
                />
                <div class="modal-actions">
                    <button type="button" on:click={saveEditLane}>Spara</button>
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

<style>
    .board-toolbar {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
        padding: 1rem 1rem 0 1rem;
        box-sizing: border-box;
    }
    .add-task-btn {
        background: none;
        border: none;
        cursor: pointer;
        padding: 0.2rem;
        display: flex;
        align-items: center;
    }
    .add-task-btn svg {
        display: block;
    }
    .board {
        display: flex;
        gap: 1rem;
        padding: 1rem;
        background: #f4f4f4;
        min-height: 60vh;
        position: relative;
    }
    .settings-btn {
        position: absolute;
        top: 1rem;
        right: 1rem;
        background: none;
        border: none;
        cursor: pointer;
        padding: 0.2rem;
        z-index: 10;
    }
    .settings-btn svg {
        display: block;
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
    .edit-lane-btn svg {
        display: block;
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
