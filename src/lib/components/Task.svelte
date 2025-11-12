<script lang="ts">
    export let task;
    export let laneId;
</script>

<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
    class="task"
    draggable="true"
    role="listitem"
    aria-grabbed="false"
    on:dragstart={(e) => {
        if (e.dataTransfer) {
            e.dataTransfer.setData(
                "text/plain",
                JSON.stringify({ taskId: task.id, fromLaneId: laneId })
            );
        }
    }}
    on:click
>
    <div>{task.title}</div>
</div>

<style>
    .task {
        background: #e0e7ef;
        border-radius: 4px;
        padding: 0.5rem 1rem;
        box-shadow: 0 1px 4px rgba(0, 0, 0, 0.03);
    }
</style>
