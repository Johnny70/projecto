<script lang="ts">
    import Task from "./Task.svelte";
    import { createEventDispatcher } from "svelte";
    export let lane;
    const dispatch = createEventDispatcher();

    let isDragOver = false;
</script>

<div
    class="swimlane {isDragOver ? 'drag-over' : ''}"
    role="list"
    aria-dropeffect="move"
    on:dragover|preventDefault={() => {
        isDragOver = true;
    }}
    on:dragleave={() => {
        isDragOver = false;
    }}
    on:drop={(e) => {
        isDragOver = false;
        if (e.dataTransfer) {
            const data = e.dataTransfer.getData("text/plain");
            if (data) {
                const { taskId, fromLaneId } = JSON.parse(data);
                dispatch("moveTask", { taskId, fromLaneId, toLaneId: lane.id });
            }
        }
    }}
>
    <!-- lane.title is rendered in Board component -->
    <div class="tasks">
        {#each lane.tasks as task (task.id)}
            <Task
                {task}
                laneId={lane.id}
                on:click={() => dispatch("taskClick", { task })}
            />
        {/each}
    </div>
</div>

<style>
    .swimlane {
        background: #fff;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
        padding: 1rem;
        min-width: 250px;
        flex: 1;
        display: flex;
        flex-direction: column;
    }
    .tasks {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
    }
    .drag-over {
        background: #e3f2fd;
        box-shadow: 0 0 0 3px #1976d2 inset;
        transition:
            background 0.2s,
            box-shadow 0.2s;
    }
</style>
