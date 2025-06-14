html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meu Checklist</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        :root {
            --primary-color: #4a6fa5;
            --secondary-color: #166088;
            --accent-color: #4fc3f7;
            --background-color: #f5f7fa;
            --text-color: #333;
            --light-text: #777;
            --border-color: #ddd;
            --high-priority: #e74c3c;
            --medium-priority: #f39c12;
            --low-priority: #2ecc71;
            --critical-priority: #9b59b6;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: var(--background-color);
            color: var(--text-color);
            line-height: 1.6;
            padding: 20px;
        }

        h1 {
            color: var(--secondary-color);
            margin-bottom: 20px;
            text-align: center;
        }

        .task-controls {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 20px;
        }

        .task-controls input, .task-controls select {
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            flex-grow: 1;
            min-width: 100%;
        }

        .task-controls button {
            padding: 10px 15px;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            min-width: 100%;
        }

        #task-list {
            list-style: none;
        }

        .task-item {
            background: white;
            padding: 15px;
            margin-bottom: 10px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .task-title {
            font-weight: bold;
            margin-bottom: 5px;
        }

        .task-priority {
            padding: 3px 8px;
            border-radius: 12px;
            font-size: 12px;
            color: white;
            display: inline-block;
            margin-bottom: 5px;
        }

        .priority-1 { background-color: var(--low-priority); }
        .priority-2 { background-color: var(--medium-priority); }
        .priority-3 { background-color: var(--high-priority); }
        .priority-4 { background-color: var(--critical-priority); }

        .task-dates {
            font-size: 12px;
            color: var(--light-text);
            margin-bottom: 10px;
        }

        .task-actions {
            display: flex;
            gap: 10px;
        }

        .task-actions button {
            padding: 5px 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 12px;
        }

        .complete-btn { background-color: #2ecc71; color: white; }
        .edit-btn { background-color: #4fc3f7; color: white; }
        .delete-btn { background-color: #e74c3c; color: white; }

        .top5-section {
            background-color: #fff8e1;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .top5-section h2 {
            color: #ff8f00;
            margin-bottom: 5px;
        }

        #top5-list {
            counter-reset: top5-counter;
            list-style: none;
        }

        #top5-list li {
            counter-increment: top5-counter;
            padding: 10px 15px 10px 35px;
            margin-bottom: 8px;
            background-color: white;
            border-radius: 4px;
            position: relative;
        }

        #top5-list li::before {
            content: counter(top5-counter);
            position: absolute;
            left: 10px;
            top: 50%;
            transform: translateY(-50%);
            width: 20px;
            height: 20px;
            background-color: var(--primary-color);
            color: white;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Meu Checklist</h1>

    <div class="top5-section">
        <h2>TOP 5 do Dia</h2>
        <ol id="top5-list"></ol>
    </div>

    <div class="task-controls">
        <input type="text" id="new-task" placeholder="Nova tarefa...">
        <select id="priority">
            <option value="1">Prioridade Baixa</option>
            <option value="2">Prioridade Média</option>
            <option value="3" selected>Prioridade Alta</option>
            <option value="4">Prioridade Crítica</option>
        </select>
        <input type="date" id="due-date">
        <button id="add-task"><i class="fas fa-plus"></i> Adicionar</button>
    </div>

    <ul id="task-list"></ul>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // Elementos do DOM
            const taskInput = document.getElementById('new-task');
            const prioritySelect = document.getElementById('priority');
            const dueDateInput = document.getElementById('due-date');
            const addTaskBtn = document.getElementById('add-task');
            const taskList = document.getElementById('task-list');
            const top5List = document.getElementById('top5-list');

            // Configurar data mínima para hoje
            const today = new Date().toISOString().split('T')[0];
            dueDateInput.min = today;
            dueDateInput.value = today;

            // Carregar tarefas do localStorage
            let tasks = JSON.parse(localStorage.getItem('tasks')) || [];

            // Função para salvar tarefas
            function saveTasks() {
                localStorage.setItem('tasks', JSON.stringify(tasks));
            }

            // Adicionar nova tarefa
            function addTask() {
                const title = taskInput.value.trim();
                const priority = parseInt(prioritySelect.value);
                const dueDate = dueDateInput.value;
                
                if (!title) {
                    alert('Por favor, insira um título para a tarefa');
                    return;
                }
                
                const newTask = {
                    id: Date.now(),
                    title,
                    priority,
                    dueDate,
                    createdAt: new Date().toISOString(),
                    completed: false,
                    completionDate: null
                };
                
                tasks.push(newTask);
                saveTasks();
                renderTasks();
                
                taskInput.value = '';
                prioritySelect.value = '3';
                dueDateInput.value = today;
                taskInput.focus();
            }

            // Renderizar tarefas
            function renderTasks() {
                taskList.innerHTML = '';
                updateTop5();
                
                if (tasks.length === 0) {
                    taskList.innerHTML = '<li>Nenhuma tarefa cadastrada</li>';
                    return;
                }
                
                tasks.forEach(task => {
                    const priorityText = 
                        task.priority === 1 ? 'Baixa' :
                        task.priority === 2 ? 'Média' :
                        task.priority === 3 ? 'Alta' : 'Crítica';
                    
                    const dueDate = new Date(task.dueDate);
                    const formattedDueDate = dueDate.toLocaleDateString('pt-BR');
                    const createdAt = new Date(task.createdAt);
                    const formattedCreatedAt = createdAt.toLocaleDateString('pt-BR');
                    
                    const taskItem = document.createElement('li');
                    taskItem.className = 'task-item';
                    if (task.completed) taskItem.classList.add('completed');
                    
                    taskItem.innerHTML = `
                        <div class="task-title">${task.title}</div>
                        <div class="task-priority priority-${task.priority}">${priorityText}</div>
                        <div class="task-dates">
                            <div>Criada em: ${formattedCreatedAt}</div>
                            <div>Prazo: ${formattedDueDate}</div>
                        </div>
                        <div class="task-actions">
                            <button class="complete-btn" data-id="${task.id}">
                                ${task.completed ? 'Desfazer' : 'Concluir'}
                            </button>
                            <button class="edit-btn" data-id="${task.id}">Editar</button>
                            <button class="delete-btn" data-id="${task.id}">Excluir</button>
                        </div>
                    `;
                    
                    taskList.appendChild(taskItem);
                });
                
                // Adicionar eventos
                document.querySelectorAll('.complete-btn').forEach(btn => {
                    btn.addEventListener('click', function() {
                        const taskId = parseInt(this.getAttribute('data-id'));
                        const task = tasks.find(task => task.id === taskId);
                        if (task) {
                            task.completed = !task.completed;
                            task.completionDate = task.completed ? new Date().toISOString() : null;
                            saveTasks();
                            renderTasks();
                        }
                    });
                });
                
                document.querySelectorAll('.edit-btn').forEach(btn => {
                    btn.addEventListener('click', function() {
                        const taskId = parseInt(this.getAttribute('data-id'));
                        const task = tasks.find(task => task.id === taskId);
                        if (task) {
                            taskInput.value = task.title;
                            prioritySelect.value = task.priority;
                            dueDateInput.value = task.dueDate;
                            
                            tasks = tasks.filter(t => t.id !== taskId);
                            saveTasks();
                            renderTasks();
                        }
                    });
                });
                
                document.querySelectorAll('.delete-btn').forEach(btn => {
                    btn.addEventListener('click', function() {
                        const taskId = parseInt(this.getAttribute('data-id'));
                        if (confirm('Tem certeza que deseja excluir esta tarefa?')) {
                            tasks = tasks.filter(task => task.id !== taskId);
                            saveTasks();
                            renderTasks();
                        }
                    });
                });
            }

            // Atualizar TOP 5
            function updateTop5() {
                top5List.innerHTML = '';
                
                const today = new Date().toISOString().split('T')[0];
                const todayTasks = tasks.filter(task => 
                    task.dueDate === today && !task.completed
                );
                
                const sortedTasks = [...todayTasks].sort((a, b) => {
                    if (b.priority !== a.priority) {
                        return b.priority - a.priority;
                    }
                    return new Date(a.createdAt) - new Date(b.createdAt);
                });
                
                const top5 = sortedTasks.slice(0, 5);
                
                if (top5.length === 0) {
                    top5List.innerHTML = '<li>Nenhuma tarefa prioritária para hoje</li>';
                    return;
                }
                
                top5.forEach(task => {
                    const priorityText = 
                        task.priority === 1 ? 'Baixa' :
                        task.priority === 2 ? 'Média' :
                        task.priority === 3 ? 'Alta' : 'Crítica';
                    
                    const taskItem = document.createElement('li');
                    taskItem.innerHTML = `
                        ${task.title} <span class="task-priority priority-${task.priority}">${priorityText}</span>
                    `;
                    
                    taskItem.addEventListener('click', () => {
                        const taskIndex = tasks.findIndex(t => t.id === task.id);
                        if (taskIndex !== -1) {
                            tasks[taskIndex].completed = true;
                            tasks[taskIndex].completionDate = new Date().toISOString();
                            saveTasks();
                            renderTasks();
                        }
                    });
                    
                    top5List.appendChild(taskItem);
                });
            }

            // Event listeners
            addTaskBtn.addEventListener('click', addTask);
            taskInput.addEventListener('keypress', function(e) {
                if (e.key === 'Enter') addTask();
            });

            // Verificar lembretes
            function checkReminders() {
                const today = new Date();
                const todayStr = today.toISOString().split('T')[0];
                const tasksDueToday = tasks.filter(task => 
                    task.dueDate === todayStr && !task.completed
                );
                
                if (tasksDueToday.length > 0) {
                    const highPriority = tasksDueToday.filter(task => task.priority >= 3).length;
                    alert(`Você tem ${tasksDueToday.length} tarefa(s) para hoje (${highPriority} prioridade alta)`);
                }
            }

            // Inicializar
            renderTasks();
            checkReminders();
        });
    </script>
</body>
</html>
