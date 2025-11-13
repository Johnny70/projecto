<script lang="ts">
    import Board from "$lib/components/Board.svelte";
    const lanes = [
        {
            id: 1,
            title: "To Do",
            tasks: [{ id: 1, title: "This is a task, click me" }],
        },
        { id: 2, title: "In Progress", tasks: [] },
        { id: 3, title: "Blocked", tasks: [] },
        { id: 4, title: "Ready for Deployment", tasks: [] },
        { id: 5, title: "Done", tasks: [] },
    ];
</script>

<img
    src="/logo.png"
    alt="Projecto logo"
    style="width: 180px; height: auto; display: block;"
/>
<Board {lanes} />
