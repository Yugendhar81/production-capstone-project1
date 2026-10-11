# production-capstone-project1

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="TaskFlow is a responsive task management application.">
    <title>TaskFlow | Task Manager</title>

    <style>
        :root {
            --primary: #4f46e5;
            --primary-dark: #3730a3;
            --background: #f5f7fb;
            --surface: #ffffff;
            --text: #1f2937;
            --muted: #64748b;
            --border: #e2e8f0;
            --success: #15803d;
            --radius: 12px;
        }

        * { box-sizing: border-box; }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: var(--background);
            color: var(--text);
            line-height: 1.6;
        }

        header {
            background: var(--primary-dark);
            color: white;
            padding: 20px;
        }

        .container {
            width: min(100% - 32px, 850px);
            margin: auto;
        }

        header h1 { margin: 0; }

        main { padding: 32px 0; }

        .panel {
            background: var(--surface);
            padding: 24px;
            border: 1px solid var(--border);
            border-radius: var(--radius);
            margin-bottom: 20px;
        }

        .task-form {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
        }

        input {
            flex: 1;
            min-width: 180px;
            padding: 12px;
            border: 1px solid #94a3b8;
            border-radius: 8px;
            font: inherit;
        }

        button {
            padding: 11px 16px;
            border: none;
            border-radius: 8px;
            background: var(--primary);
            color: white;
            font: inherit;
            cursor: pointer;
        }

        button:hover { background: var(--primary-dark); }

        button:focus-visible,
        input:focus-visible {
            outline: 3px solid #f59e0b;
            outline-offset: 3px;
        }

        .task-list {
            list-style: none;
            padding: 0;
            margin: 0;
        }

        .task {
            display: flex;
            align-items: center;
            gap: 12px;
            padding: 14px 0;
            border-bottom: 1px solid var(--border);
        }

        .task:last-child { border-bottom: none; }

        .task-title {
            flex: 1;
            overflow-wrap: anywhere;
        }

        .task.completed .task-title {
            text-decoration: line-through;
            color: var(--muted);
        }

        .task input[type="checkbox"] {
            flex: 0 0 auto;
            min-width: auto;
            width: 18px;
            height: 18px;
        }

        .delete-button { background: #b91c1c; }

        .delete-button:hover { background: #991b1b; }

        .stats {
            color: var(--muted);
            font-size: 0.95rem;
        }

        #status {
            min-height: 24px;
            color: var(--success);
        }

        footer {
            padding: 24px;
            text-align: center;
            color: var(--muted);
        }

        @media (max-width: 480px) {
            .panel { padding: 16px; }

            .task-form button { width: 100%; }

            .task {
                align-items: flex-start;
                flex-wrap: wrap;
            }
        }
    </style>
</head>

<body>
    <header>
        <div class="container">
            <h1>TaskFlow</h1>
            <p>Your simple personal task manager</p>
        </div>
    </header>

    <main class="container">
        <section class="panel" aria-labelledby="add-heading">
            <h2 id="add-heading">Add a task</h2>

            <form id="task-form" class="task-form">
                <label for="task-input">Task description</label>
                <input
                    id="task-input"
                    type="text"
                    maxlength="200"
                    placeholder="What do you need to do?"
                    required
                >
                <button type="submit">Add Task</button>
            </form>

            <p id="status" role="status" aria-live="polite"></p>
        </section>

        <section class="panel" aria-labelledby="list-heading">
            <h2 id="list-heading">My Tasks</h2>
            <p id="stats" class="stats">0 tasks</p>
            <ul id="task-list" class="task-list"></ul>
            <p id="empty-message">No tasks yet. Add your first task above!</p>
        </section>
    </main>

    <footer>
        <p>TaskFlow — Production Capstone Project</p>
    </footer>

    <script>
        const form = document.getElementById("task-form");
        const input = document.getElementById("task-input");
        const list = document.getElementById("task-list");
        const stats = document.getElementById("stats");
        const status = document.getElementById("status");
        const emptyMessage = document.getElementById("empty-message");

        let tasks = [];

        try {
            const saved = JSON.parse(
                localStorage.getItem("taskflow-tasks") || "[]"
            );

            if (Array.isArray(saved)) {
                tasks = saved.filter(task =>
                    task &&
                    typeof task.id === "string" &&
                    typeof task.title === "string" &&
                    typeof task.completed === "boolean"
                );
            }
        } catch {
            tasks = [];
        }

        function saveTasks() {
            try {
                localStorage.setItem("taskflow-tasks", JSON.stringify(tasks));
                return true;
            } catch {
                status.textContent =
                    "Changes may not persist because browser storage is unavailable.";
                return false;
            }
        }

        function renderTasks() {
            list.replaceChildren();

            tasks.forEach(task => {
                const item = document.createElement("li");
                item.className = task.completed
                    ? "task completed"
                    : "task";

                const checkbox = document.createElement("input");
                checkbox.type = "checkbox";
                checkbox.checked = task.completed;
                checkbox.setAttribute(
                    "aria-label",
                    `Mark ${task.title} as completed`
                );

                checkbox.addEventListener("change", () => {
                    task.completed = checkbox.checked;
                    saveTasks();
                    renderTasks();
                    status.textContent = task.completed
                        ? "Task completed."
                        : "Task marked as incomplete.";
                });

                const title = document.createElement("span");
                title.className = "task-title";
                title.textContent = task.title;

                const remove = document.createElement("button");
                remove.type = "button";
                remove.className = "delete-button";
                remove.textContent = "Delete";
                remove.setAttribute("aria-label", `Delete ${task.title}`);

                remove.addEventListener("click", () => {
                    tasks = tasks.filter(item => item.id !== task.id);
                    saveTasks();
                    renderTasks();
                    status.textContent = "Task deleted.";
                });

                item.append(checkbox, title, remove);
                list.append(item);
            });

            const completed = tasks.filter(task => task.completed).length;
            stats.textContent =
                `${tasks.length} tasks · ${completed} completed`;
            emptyMessage.hidden = tasks.length > 0;
        }

        form.addEventListener("submit", event => {
            event.preventDefault();

            const title = input.value.trim();

            if (!title) {
                status.textContent = "Please enter a task description.";
                input.focus();
                return;
            }

            tasks.push({
                id: crypto.randomUUID(),
                title,
                completed: false
            });

            saveTasks();
            renderTasks();
            form.reset();
            input.focus();
            status.textContent = "Task added successfully.";
        });

        renderTasks();
    </script>
</body>
</html>
