# Projecto – Svelte Trello-like Board

A clean, modern project management board built with SvelteKit. Features swimlanes, draggable tasks, modals for editing, and persistent localStorage data. Ready for GitHub!

## Features

-   Board with swimlanes and tasks
-   Add, edit, and remove swimlanes
-   Add, view, and remove tasks
-   Drag-and-drop tasks between swimlanes
-   Persistent state in localStorage
-   Accessible modals and strict TypeScript

## Getting Started

1. **Install dependencies:**

    ```sh
    npm install
    ```

2. **Start development server:**

    ```sh
    npm run dev
    ```

3. **Build for production:**

    ```sh
    npm run build
    ```

4. **Preview production build:**

    ```sh
    npm run preview
    ```

## Folder Structure

-   `src/lib/components/Board.svelte` – Main board logic
-   `src/lib/components/Swimlane.svelte` – Swimlane rendering
-   `src/lib/components/Task.svelte` – Task rendering
-   `src/lib/index.ts` – Library entry

## Contributing

Pull requests and issues are welcome!

## License

MIT
