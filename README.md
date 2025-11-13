# Projecto – Svelte Task Board with Swimlanes

A clean, modern project management board built with SvelteKit. Features swimlanes, draggable tasks, modals for editing, and persistent localStorage data.

## Features

-   Basic drag-and-drop (desktop) – can be extended for mobile and advanced feedback
-   Board with swimlanes and tasks
-   Add, edit, and remove swimlanes
-   Add, edit, view, and remove tasks
-   Modal dialogs for editing swimlanes and tasks
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

-   The codebase is clean, modular and easy to extend for new features (mobile support, tests, etc)

Pull requests and issues are welcome!

## License

GPL-3.0

## Author

Created by Johnny70 (johnny.jakobsson@gmail.com)