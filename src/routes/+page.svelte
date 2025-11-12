<script lang="ts">
    import Board from "$lib/components/Board.svelte";
    const lanes = [
        {
            id: 1,
            title: "Att göra",
            tasks: [
                { id: 1, title: "Skapa projektstruktur" },
                { id: 2, title: "Designa UI" },
            ],
        },
        {
            id: 2,
            title: "Pågår",
            tasks: [{ id: 3, title: "Implementera Board-komponent" }],
        },
        {
            id: 3,
            title: "Klart",
            tasks: [{ id: 4, title: "Skapa Task-komponent" }],
        },
    ];
</script>

<h1>Projektboard</h1>
<Board {lanes} />
