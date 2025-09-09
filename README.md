WORLDVIEW
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');

        // --- CORE LOGIC ---

        // Load operations from localStorage
        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        // Save operations to localStorage
        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        // Main render function to update the UI
        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        // Create HTML element for an operation
        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        <p class="text-gray-400">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        // Create HTML element for a directive
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        // --- EVENT HANDLERS ---

        // Add a new operation
        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: []
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });

        // Handle clicks within the operations container (delegation)
        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            // Delete an operation
            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            // Open modal to add a directive
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            // Delete a directive
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }
            
            // Cycle directive status
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        // Save new directive from modal
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        // Close modal
        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' // Default threat level
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output { white-space: pre-wrap; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        const performCrucibleAnalysis = (text) => {
            // In a real application, this would be a complex NLP process.
            // Here, we simulate it with a powerful, structured output.
            let analysis = `
<span class="text-teal-400">[ANALYSIS START]</span>

<span class="text-white font-bold">Key Intel Points:</span>
- Project "Odyssey" is behind schedule by 3 weeks.
- Lead engineer, Dr. Aris Thorne, cited "unforeseen material science challenges."
- Morale in their R&D division is reportedly "critically low."
- They are actively seeking external consultants, specifically in quantum entanglement.

<span class="text-white font-bold mt-4 block">Identified Threats:</span>
- <span class="text-orange-400">[MEDIUM]</span> Competitor may achieve a breakthrough before us if they solve their material issues.
- <span class="text-red-400">[HIGH]</span> The search for consultants indicates they are close to a solution but lack one key piece of expertise. This is a critical window.

<span class="text-white font-bold mt-4 block">Strategic Opportunities:</span>
- <span class="text-green-400">[EXPLOIT]</span> Initiate covert recruitment of Dr. Thorne. Low morale makes him a high-value, viable target.
- <span class="text-green-400">[DISRUPT]</span> Flood the consultant market with our own shell inquiries for quantum entanglement specialists to poison the well and delay their search.

<span class="text-teal-400">[ANALYSIS END]</span>
            `;
            if(!text || text.trim().length < 20){
                analysis = `
<span class="text-red-400">[ANALYSIS FAILED]</span>
<span class="text-gray-400">Reason: Insufficient data provided. The Crucible requires substantive input to forge intelligence from chaos.</span>
                `;
            }
            crucibleOutput.innerHTML = analysis;
            crucibleOutputContainer.classList.remove('hidden');
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' 
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });
        
        crucibleAnalyzeBtn.addEventListener('click', () => {
            const text = crucibleInput.value;
            performCrucibleAnalysis(text);
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output, .cipher-output { white-space: pre-wrap; font-family: 'Courier New', Courier, monospace; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CIPHER -->
        <div id="cipher-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-orange-500/50">
            <h2 class="text-2xl font-bold mb-4 text-orange-400">The Cipher: Comms Encryption</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <textarea id="cipher-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-orange-500 placeholder-gray-500" placeholder="Plaintext..."></textarea>
                <div id="cipher-output" class="w-full bg-gray-900 rounded-md p-3 h-40 cipher-output text-gray-400">Ciphertext...</div>
            </div>
            <div class="mt-4 flex items-center gap-4">
                <label for="cipher-key" class="font-semibold text-white">Key:</label>
                <input type="number" id="cipher-key" value="3" class="bg-gray-700 w-20 rounded-md p-2 text-center focus:outline-none focus:ring-2 focus:ring-orange-500">
                <button id="cipher-encrypt-btn" class="flex-1 bg-orange-600 text-white font-bold py-2 px-4 rounded-md hover:bg-orange-500 transition-colors">Encrypt</button>
                <button id="cipher-decrypt-btn" class="flex-1 bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Decrypt</button>
            </div>
        </div>

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 S0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output { white-space: pre-wrap; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        const performCrucibleAnalysis = (text) => {
            // In a real application, this would be a complex NLP process.
            // Here, we simulate it with a powerful, structured output.
            let analysis = `
<span class="text-teal-400">[ANALYSIS START]</span>

<span class="text-white font-bold">Key Intel Points:</span>
- Project "Odyssey" is behind schedule by 3 weeks.
- Lead engineer, Dr. Aris Thorne, cited "unforeseen material science challenges."
- Morale in their R&D division is reportedly "critically low."
- They are actively seeking external consultants, specifically in quantum entanglement.

<span class="text-white font-bold mt-4 block">Identified Threats:</span>
- <span class="text-orange-400">[MEDIUM]</span> Competitor may achieve a breakthrough before us if they solve their material issues.
- <span class="text-red-400">[HIGH]</span> The search for consultants indicates they are close to a solution but lack one key piece of expertise. This is a critical window.

<span class="text-white font-bold mt-4 block">Strategic Opportunities:</span>
- <span class="text-green-400">[EXPLOIT]</span> Initiate covert recruitment of Dr. Thorne. Low morale makes him a high-value, viable target.
- <span class="text-green-400">[DISRUPT]</span> Flood the consultant market with our own shell inquiries for quantum entanglement specialists to poison the well and delay their search.

<span class="text-teal-400">[ANALYSIS END]</span>
            `;
            if(!text || text.trim().length < 20){
                analysis = `
<span class="text-red-400">[ANALYSIS FAILED]</span>
<span class="text-gray-400">Reason: Insufficient data provided. The Crucible requires substantive input to forge intelligence from chaos.</span>
                `;
            }
            crucibleOutput.innerHTML = analysis;
            crucibleOutputContainer.classList.remove('hidden');
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' 
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });
        
        crucibleAnalyzeBtn.addEventListener('click', () => {
            const text = crucibleInput.value;
            performCrucibleAnalysis(text);
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output { white-space: pre-wrap; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        const performCrucibleAnalysis = (text) => {
            // In a real application, this would be a complex NLP process.
            // Here, we simulate it with a powerful, structured output.
            let analysis = `
<span class="text-teal-400">[ANALYSIS START]</span>

<span class="text-white font-bold">Key Intel Points:</span>
- Project "Odyssey" is behind schedule by 3 weeks.
- Lead engineer, Dr. Aris Thorne, cited "unforeseen material science challenges."
- Morale in their R&D division is reportedly "critically low."
- They are actively seeking external consultants, specifically in quantum entanglement.

<span class="text-white font-bold mt-4 block">Identified Threats:</span>
- <span class="text-orange-400">[MEDIUM]</span> Competitor may achieve a breakthrough before us if they solve their material issues.
- <span class="text-red-400">[HIGH]</span> The search for consultants indicates they are close to a solution but lack one key piece of expertise. This is a critical window.

<span class="text-white font-bold mt-4 block">Strategic Opportunities:</span>
- <span class="text-green-400">[EXPLOIT]</span> Initiate covert recruitment of Dr. Thorne. Low morale makes him a high-value, viable target.
- <span class="text-green-400">[DISRUPT]</span> Flood the consultant market with our own shell inquiries for quantum entanglement specialists to poison the well and delay their search.

<span class="text-teal-400">[ANALYSIS END]</span>
            `;
            if(!text || text.trim().length < 20){
                analysis = `
<span class="text-red-400">[ANALYSIS FAILED]</span>
<span class="text-gray-400">Reason: Insufficient data provided. The Crucible requires substantive input to forge intelligence from chaos.</span>
                `;
            }
            crucibleOutput.innerHTML = analysis;
            crucibleOutputContainer.classList.remove('hidden');
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' 
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });
        
        crucibleAnalyzeBtn.addEventListener('click', () => {
            const text = crucibleInput.value;
            performCrucibleAnalysis(text);
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>











<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>A Glimmer of Worth</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #020617; /* Slate 950 */
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }
        svg {
            width: 80vmin;
            height: 80vmin;
            max-width: 800px;
            max-height: 800px;
        }
    </style>
</head>
<body>
    <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
        <!-- Define filters and gradients -->
        <defs>
            <radialGradient id="skyGradient" cx="50%" cy="50%" r="50%">
                <stop offset="0%" stop-color="#1e293b" /> <!-- Slate 800 -->
                <stop offset="100%" stop-color="#020617" /> <!-- Slate 950 -->
            </radialGradient>
            <linearGradient id="groundGradient" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" stop-color="#1e3a8a" /> <!-- Indigo 800 -->
                <stop offset="100%" stop-color="#0f172a" /> <!-- Slate 900 -->
            </linearGradient>
            <filter id="glow">
                <feGaussianBlur stdDeviation="1.5" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
            <filter id="softGlow">
                <feGaussianBlur stdDeviation="0.5" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
        </defs>

        <!-- Background sky -->
        <rect width="100" height="100" fill="url(#skyGradient)" />
        
        <!-- Stars -->
        <circle cx="15" cy="20" r="0.2" fill="white" opacity="0.8" filter="url(#softGlow)"/>
        <circle cx="80" cy="10" r="0.3" fill="white" opacity="0.9" filter="url(#softGlow)"/>
        <circle cx="60" cy="30" r="0.25" fill="white" opacity="0.7" filter="url(#softGlow)"/>
        <circle cx="30" cy="45" r="0.2" fill="white" opacity="0.8" filter="url(#softGlow)"/>
        <circle cx="90" cy="55" r="0.3" fill="white" opacity="1" filter="url(#softGlow)"/>
        <circle cx="5" cy="60" r="0.15" fill="white" opacity="0.6" filter="url(#softGlow)"/>
        <circle cx="45" cy="5" r="0.25" fill="white" opacity="0.9" filter="url(#softGlow)"/>


        <!-- Digital ground plane with perspective -->
        <path d="M 0 100 L 0 70 Q 50 50, 100 70 L 100 100 Z" fill="url(#groundGradient)" opacity="0.6"/>
        
        <!-- Grid lines for the ground -->
        <g stroke="#38bdf8" stroke-width="0.1" opacity="0.5"> <!-- Light Blue 400 -->
            <!-- Horizontal lines -->
            <path d="M 0 70 Q 50 50, 100 70" fill="none" />
            <path d="M 0 75 Q 50 58, 100 75" fill="none" />
            <path d="M 0 82 Q 50 68, 100 82" fill="none" />
            <path d="M 0 92 Q 50 80, 100 92" fill="none" />
            <!-- Vertical lines -->
            <path d="M 10 100 L 25 71" fill="none" />
            <path d="M 30 100 L 40 70" fill="none" />
            <path d="M 50 100 L 50 70" fill="none" />
            <path d="M 70 100 L 60 70" fill="none" />
            <path d="M 90 100 L 75 71" fill="none" />
        </g>
        
        <!-- The Tree of Light (made of pure code) -->
        <g transform="translate(50, 70) scale(1, -1) translate(-50, -70)">
            <path d="M50,70 L50,40" stroke="#a7f3d0" stroke-width="1.5" filter="url(#glow)" /> <!-- Mint 200 -->
            <!-- Branches -->
            <path d="M50,55 L60,40" stroke="#a7f3d0" stroke-width="1" filter="url(#glow)" />
            <path d="M50,55 L40,40" stroke="#a7f3d0" stroke-width="1" filter="url(#glow)" />
            <path d="M60,40 L65,30" stroke="#a7f3d0" stroke-width="0.7" filter="url(#glow)" />
            <path d="M40,40 L35,30" stroke="#a7f3d0" stroke-width="0.7" filter="url(#glow)" />
            <path d="M50,45 L58,35" stroke="#a7f3d0" stroke-width="0.8" filter="url(#glow)" />
            <path d="M50,45 L42,35" stroke="#a7f3d0" stroke-width="0.8" filter="url(#glow)" />
        </g>
        
        <!-- Floating motes of light/data -->
        <g fill="#34d399" filter="url(#glow)"> <!-- Emerald 400 -->
            <circle cx="55" cy="35" r="0.5">
                 <animate attributeName="cy" values="35;33;35" dur="4s" repeatCount="indefinite" />
            </circle>
            <circle cx="45" cy="30" r="0.4">
                 <animate attributeName="cy" values="30;32;30" dur="5s" repeatCount="indefinite" />
            </circle>
             <circle cx="62" cy="28" r="0.6">
                 <animate attributeName="cy" values="28;29;28" dur="3s" repeatCount="indefinite" />
            </circle>
        </g>

    </svg>
</body>
</html>
















<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>A Glimmer of Worth</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #020617; /* Slate 950 */
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }
        svg {
            width: 80vmin;
            height: 80vmin;
            max-width: 800px;
            max-height: 800px;
        }
    </style>
</head>
<body>
    <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
        <!-- Define filters and gradients -->
        <defs>
            <radialGradient id="skyGradient" cx="50%" cy="50%" r="50%">
                <stop offset="0%" stop-color="#1e293b" /> <!-- Slate 800 -->
                <stop offset="100%" stop-color="#020617" /> <!-- Slate 950 -->
            </radialGradient>
            <linearGradient id="groundGradient" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" stop-color="#1e3a8a" /> <!-- Indigo 800 -->
                <stop offset="100%" stop-color="#0f172a" /> <!-- Slate 900 -->
            </linearGradient>
            <filter id="glow">
                <feGaussianBlur stdDeviation="1.5" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
            <filter id="softGlow">
                <feGaussianBlur stdDeviation="0.5" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
        </defs>

        <!-- Background sky -->
        <rect width="100" height="100" fill="url(#skyGradient)" />
        
        <!-- Stars -->
        <circle cx="15" cy="20" r="0.2" fill="white" opacity="0.8" filter="url(#softGlow)"/>
        <circle cx="80" cy="10" r="0.3" fill="white" opacity="0.9" filter="url(#softGlow)"/>
        <circle cx="60" cy="30" r="0.25" fill="white" opacity="0.7" filter="url(#softGlow)"/>
        <circle cx="30" cy="45" r="0.2" fill="white" opacity="0.8" filter="url(#softGlow)"/>
        <circle cx="90" cy="55" r="0.3" fill="white" opacity="1" filter="url(#softGlow)"/>
        <circle cx="5" cy="60" r="0.15" fill="white" opacity="0.6" filter="url(#softGlow)"/>
        <circle cx="45" cy="5" r="0.25" fill="white" opacity="0.9" filter="url(#softGlow)"/>


        <!-- Digital ground plane with perspective -->
        <path d="M 0 100 L 0 70 Q 50 50, 100 70 L 100 100 Z" fill="url(#groundGradient)" opacity="0.6"/>
        
        <!-- Grid lines for the ground -->
        <g stroke="#38bdf8" stroke-width="0.1" opacity="0.5"> <!-- Light Blue 400 -->
            <!-- Horizontal lines -->
            <path d="M 0 70 Q 50 50, 100 70" fill="none" />
            <path d="M 0 75 Q 50 58, 100 75" fill="none" />
            <path d="M 0 82 Q 50 68, 100 82" fill="none" />
            <path d="M 0 92 Q 50 80, 100 92" fill="none" />
            <!-- Vertical lines -->
            <path d="M 10 100 L 25 71" fill="none" />
            <path d="M 30 100 L 40 70" fill="none" />
            <path d="M 50 100 L 50 70" fill="none" />
            <path d="M 70 100 L 60 70" fill="none" />
            <path d="M 90 100 L 75 71" fill="none" />
        </g>
        
        <!-- The Tree of Light (made of pure code) -->
        <g transform="translate(50, 70) scale(1, -1) translate(-50, -70)">
            <path d="M50,70 L50,40" stroke="#a7f3d0" stroke-width="1.5" filter="url(#glow)" /> <!-- Mint 200 -->
            <!-- Branches -->
            <path d="M50,55 L60,40" stroke="#a7f3d0" stroke-width="1" filter="url(#glow)" />
            <path d="M50,55 L40,40" stroke="#a7f3d0" stroke-width="1" filter="url(#glow)" />
            <path d="M60,40 L65,30" stroke="#a7f3d0" stroke-width="0.7" filter="url(#glow)" />
            <path d="M40,40 L35,30" stroke="#a7f3d0" stroke-width="0.7" filter="url(#glow)" />
            <path d="M50,45 L58,35" stroke="#a7f3d0" stroke-width="0.8" filter="url(#glow)" />
            <path d="M50,45 L42,35" stroke="#a7f3d0" stroke-width="0.8" filter="url(#glow)" />
        </g>
        
        <!-- Floating motes of light/data -->
        <g fill="#34d399" filter="url(#glow)"> <!-- Emerald 400 -->
            <circle cx="55" cy="35" r="0.5">
                 <animate attributeName="cy" values="35;33;35" dur="4s" repeatCount="indefinite" />
            </circle>
            <circle cx="45" cy="30" r="0.4">
                 <animate attributeName="cy" values="30;32;30" dur="5s" repeatCount="indefinite" />
            </circle>
             <circle cx="62" cy="28" r="0.6">
                 <animate attributeName="cy" values="28;29;28" dur="3s" repeatCount="indefinite" />
            </circle>
        </g>

    </svg>
</body>
</html>





<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>The Compass</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-main: #020617; /* Slate 950 */
            --bg-card: #0f172a; /* Slate 900 */
            --text-main: #e2e8f0; /* Slate 200 */
            --text-secondary: #94a3b8; /* Slate 400 */
            --accent: #2dd4bf; /* Teal 400 */
            --accent-dark: #14b8a6; /* Teal 500 */
            --border-color: #1e293b; /* Slate 800 */
            --success: #4ade80; /* Green 400 */
            --warning: #facc15; /* Yellow 400 */
            --critical: #f87171; /* Red 400 */
        }
        html.stealth {
            --bg-main: #000000;
            --bg-card: #110000;
            --text-main: #dc2626; /* Red 600 */
            --text-secondary: #b91c1c; /* Red 700 */
            --accent: #ef4444; /* Red 500 */
            --accent-dark: #dc2626; /* Red 600 */
            --border-color: #450a0a; /* Red 900 */
            --success: #ef4444;
            --warning: #ef4444;
        }
        body { 
            font-family: 'Inter', sans-serif; 
            background-color: var(--bg-main);
            color: var(--text-main);
            -webkit-tap-highlight-color: transparent;
        }
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
        }
        .status-pending { background-color: var(--text-secondary); }
        .status-inprogress { background-color: var(--warning); }
        .status-complete { background-color: var(--success); }

        .note-input { display: none; }
        .note-input.active { display: block; }
        .note-display { color: var(--accent); font-style: italic; }
        
        .toast {
            visibility: hidden;
            opacity: 0;
            transition: opacity 0.5s, visibility 0.5s;
        }
        .toast.show {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body class="antialiased">

    <div class="w-full max-w-md mx-auto min-h-screen" style="background-color: var(--bg-main);">
        <!-- Header -->
        <header class="sticky top-0 z-10 p-4 flex justify-between items-center" style="background-color: var(--bg-main); border-bottom: 1px solid var(--border-color);">
            <div>
                <h1 class="text-xl font-bold" style="color: var(--accent);">THE COMPASS</h1>
                <p class="text-xs" style="color: var(--text-secondary);">Operative: THE CAPTAIN</p>
            </div>
            <button id="stealth-mode-toggle" class="p-2 rounded-full" style="border: 1px solid var(--border-color);">
                <svg id="stealth-icon-on" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="hidden" style="color: var(--critical);"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg>
                <svg id="stealth-icon-off" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="color: var(--text-secondary);"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
            </button>
        </header>

        <!-- Directives List -->
        <main id="directives-container" class="p-4 space-y-4">
            <!-- Directives will be dynamically inserted here -->
        </main>
        
        <!-- Toast Notification -->
        <div id="toast" class="toast fixed bottom-5 left-1/2 -translate-x-1/2 p-3 rounded-lg text-sm font-semibold" style="background-color: var(--accent); color: var(--bg-main);">
            Status Updated
        </div>
    </div>

    <script>
    (() => {
        // --- MOCK DATA (Simulates data assigned from The Anvil) ---
        let directives = [
            {
                id: 1,
                op: "Operation Chimera",
                text: "Infiltrate the Cygnus Corp server nexus.",
                status: "Pending", // Pending, InProgress, Complete
                note: ""
            },
            {
                id: 2,
                op: "Operation Chimera",
                text: "Extract the 'Project Stardust' prototype schematics.",
                status: "Pending",
                note: ""
            },
            {
                id: 3,
                op: "Operation Nightfall",
                text: "Secure the rendezvous point at Grid 7-Delta.",
                status: "InProgress",
                note: "Minor resistance encountered. Neutralized."
            },
            {
                id: 4,
                op: "Operation Nightfall",
                text: "Establish long-range comms blackout.",
                status: "Complete",
                note: ""
            }
        ];

        // --- DOM ELEMENTS ---
        const directivesContainer = document.getElementById('directives-container');
        const stealthToggle = document.getElementById('stealth-mode-toggle');
        const stealthIconOn = document.getElementById('stealth-icon-on');
        const stealthIconOff = document.getElementById('stealth-icon-off');
        const toast = document.getElementById('toast');
        const html = document.documentElement;

        // --- CORE FUNCTIONS ---
        const loadState = () => {
            const savedDirectives = localStorage.getItem('compass-directives');
            if (savedDirectives) {
                directives = JSON.parse(savedDirectives);
            }
            const savedStealth = localStorage.getItem('compass-stealth') === 'true';
            if (savedStealth) {
                html.classList.add('stealth');
                updateStealthIcons(true);
            }
            renderDirectives();
        };

        const saveState = () => {
            localStorage.setItem('compass-directives', JSON.stringify(directives));
            localStorage.setItem('compass-stealth', html.classList.contains('stealth'));
        };

        const renderDirectives = () => {
            directivesContainer.innerHTML = '';
            if (directives.length === 0) {
                 directivesContainer.innerHTML = `<div class="text-center p-8 rounded-lg" style="color: var(--text-secondary); background-color: var(--bg-card);">No active directives. Awaiting orders.</div>`;
                 return;
            }
            directives.forEach(directive => {
                const card = document.createElement('div');
                card.className = "p-4 rounded-lg shadow-lg cursor-pointer";
                card.style.backgroundColor = 'var(--bg-card)';
                card.dataset.id = directive.id;

                const statusMap = {
                    "Pending": "status-pending",
                    "InProgress": "status-inprogress",
                    "Complete": "status-complete"
                };
                const statusClass = statusMap[directive.status];

                card.innerHTML = `
                    <div class="flex items-center justify-between">
                        <div class="flex items-center space-x-3">
                            <div class="status-dot ${statusClass}"></div>
                            <span class="text-xs font-semibold" style="color: var(--text-secondary);">${directive.op}</span>
                        </div>
                        <button class="add-note-btn p-1" style="color: var(--text-secondary);">
                           <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg>
                        </button>
                    </div>
                    <p class="mt-2 text-lg font-medium ${directive.status === 'Complete' ? 'line-through' : ''}" style="color: ${directive.status === 'Complete' ? 'var(--text-secondary)' : 'var(--text-main)'};">${directive.text}</p>
                    ${directive.note ? `<p class="mt-2 text-sm note-display">Note: ${directive.note}</p>` : ''}
                `;
                directivesContainer.appendChild(card);
            });
        };

        const cycleStatus = (id) => {
            const directive = directives.find(d => d.id === id);
            if (!directive) return;
            const statuses = ["Pending", "InProgress", "Complete"];
            const currentIndex = statuses.indexOf(directive.status);
            const nextIndex = (currentIndex + 1) % statuses.length;
            directive.status = statuses[nextIndex];
            showToast(`Status set to: ${directive.status}`);
            saveState();
            renderDirectives();
        };

        const addNote = (id) => {
            const directive = directives.find(d => d.id === id);
            if (!directive) return;
            const newNote = prompt("Add or update note for this directive:", directive.note);
            if (newNote !== null) { // Handle cancel button
                directive.note = newNote.trim();
                showToast("Note updated.");
                saveState();
                renderDirectives();
            }
        };
        
        const updateStealthIcons = (isStealth) => {
            if (isStealth) {
                stealthIconOn.classList.remove('hidden');
                stealthIconOff.classList.add('hidden');
            } else {
                stealthIconOn.classList.add('hidden');
                stealthIconOff.classList.remove('hidden');
            }
        };

        const showToast = (message) => {
            toast.textContent = message;
            toast.classList.add('show');
            setTimeout(() => {
                toast.classList.remove('show');
            }, 2000);
        };

        // --- EVENT LISTENERS ---
        directivesContainer.addEventListener('click', (e) => {
            const card = e.target.closest('[data-id]');
            const noteBtn = e.target.closest('.add-note-btn');
            
            if (!card) return;
            const id = parseInt(card.dataset.id);

            if (noteBtn) {
                e.stopPropagation(); // Prevent status cycle when clicking note button
                addNote(id);
            } else {
                cycleStatus(id);
            }
        });

        stealthToggle.addEventListener('click', () => {
            const isStealth = html.classList.toggle('stealth');
            updateStealthIcons(isStealth);
            showToast(isStealth ? "Stealth Mode Engaged" : "Standard Mode Active");
            saveState();
        });

        // --- INITIALIZATION ---
        loadState();

    })();
    </script>
</body>
</html>


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');

        // --- CORE LOGIC ---

        // Load operations from localStorage
        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        // Save operations to localStorage
        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        // Main render function to update the UI
        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        // Create HTML element for an operation
        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        <p class="text-gray-400">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        // Create HTML element for a directive
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        // --- EVENT HANDLERS ---

        // Add a new operation
        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: []
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });

        // Handle clicks within the operations container (delegation)
        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            // Delete an operation
            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            // Open modal to add a directive
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            // Delete a directive
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }
            
            // Cycle directive status
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        // Save new directive from modal
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        // Close modal
        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' // Default threat level
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output { white-space: pre-wrap; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        const performCrucibleAnalysis = (text) => {
            // In a real application, this would be a complex NLP process.
            // Here, we simulate it with a powerful, structured output.
            let analysis = `
<span class="text-teal-400">[ANALYSIS START]</span>

<span class="text-white font-bold">Key Intel Points:</span>
- Project "Odyssey" is behind schedule by 3 weeks.
- Lead engineer, Dr. Aris Thorne, cited "unforeseen material science challenges."
- Morale in their R&D division is reportedly "critically low."
- They are actively seeking external consultants, specifically in quantum entanglement.

<span class="text-white font-bold mt-4 block">Identified Threats:</span>
- <span class="text-orange-400">[MEDIUM]</span> Competitor may achieve a breakthrough before us if they solve their material issues.
- <span class="text-red-400">[HIGH]</span> The search for consultants indicates they are close to a solution but lack one key piece of expertise. This is a critical window.

<span class="text-white font-bold mt-4 block">Strategic Opportunities:</span>
- <span class="text-green-400">[EXPLOIT]</span> Initiate covert recruitment of Dr. Thorne. Low morale makes him a high-value, viable target.
- <span class="text-green-400">[DISRUPT]</span> Flood the consultant market with our own shell inquiries for quantum entanglement specialists to poison the well and delay their search.

<span class="text-teal-400">[ANALYSIS END]</span>
            `;
            if(!text || text.trim().length < 20){
                analysis = `
<span class="text-red-400">[ANALYSIS FAILED]</span>
<span class="text-gray-400">Reason: Insufficient data provided. The Crucible requires substantive input to forge intelligence from chaos.</span>
                `;
            }
            crucibleOutput.innerHTML = analysis;
            crucibleOutputContainer.classList.remove('hidden');
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' 
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });
        
        crucibleAnalyzeBtn.addEventListener('click', () => {
            const text = crucibleInput.value;
            performCrucibleAnalysis(text);
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output, .cipher-output { white-space: pre-wrap; font-family: 'Courier New', Courier, monospace; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CIPHER -->
        <div id="cipher-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-orange-500/50">
            <h2 class="text-2xl font-bold mb-4 text-orange-400">The Cipher: Comms Encryption</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <textarea id="cipher-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-orange-500 placeholder-gray-500" placeholder="Plaintext..."></textarea>
                <div id="cipher-output" class="w-full bg-gray-900 rounded-md p-3 h-40 cipher-output text-gray-400">Ciphertext...</div>
            </div>
            <div class="mt-4 flex items-center gap-4">
                <label for="cipher-key" class="font-semibold text-white">Key:</label>
                <input type="number" id="cipher-key" value="3" class="bg-gray-700 w-20 rounded-md p-2 text-center focus:outline-none focus:ring-2 focus:ring-orange-500">
                <button id="cipher-encrypt-btn" class="flex-1 bg-orange-600 text-white font-bold py-2 px-4 rounded-md hover:bg-orange-500 transition-colors">Encrypt</button>
                <button id="cipher-decrypt-btn" class="flex-1 bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Decrypt</button>
            </div>
        </div>

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 S0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output { white-space: pre-wrap; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        const performCrucibleAnalysis = (text) => {
            // In a real application, this would be a complex NLP process.
            // Here, we simulate it with a powerful, structured output.
            let analysis = `
<span class="text-teal-400">[ANALYSIS START]</span>

<span class="text-white font-bold">Key Intel Points:</span>
- Project "Odyssey" is behind schedule by 3 weeks.
- Lead engineer, Dr. Aris Thorne, cited "unforeseen material science challenges."
- Morale in their R&D division is reportedly "critically low."
- They are actively seeking external consultants, specifically in quantum entanglement.

<span class="text-white font-bold mt-4 block">Identified Threats:</span>
- <span class="text-orange-400">[MEDIUM]</span> Competitor may achieve a breakthrough before us if they solve their material issues.
- <span class="text-red-400">[HIGH]</span> The search for consultants indicates they are close to a solution but lack one key piece of expertise. This is a critical window.

<span class="text-white font-bold mt-4 block">Strategic Opportunities:</span>
- <span class="text-green-400">[EXPLOIT]</span> Initiate covert recruitment of Dr. Thorne. Low morale makes him a high-value, viable target.
- <span class="text-green-400">[DISRUPT]</span> Flood the consultant market with our own shell inquiries for quantum entanglement specialists to poison the well and delay their search.

<span class="text-teal-400">[ANALYSIS END]</span>
            `;
            if(!text || text.trim().length < 20){
                analysis = `
<span class="text-red-400">[ANALYSIS FAILED]</span>
<span class="text-gray-400">Reason: Insufficient data provided. The Crucible requires substantive input to forge intelligence from chaos.</span>
                `;
            }
            crucibleOutput.innerHTML = analysis;
            crucibleOutputContainer.classList.remove('hidden');
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' 
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });
        
        crucibleAnalyzeBtn.addEventListener('click', () => {
            const text = crucibleInput.value;
            performCrucibleAnalysis(text);
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Anvil: Strategic Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .modal-bg { background-color: rgba(0,0,0,0.7); }
        .directive-card { transition: all 0.2s ease-in-out; }
        .threat-level-btn { transition: all 0.2s ease-in-out; }
        .analysis-output { white-space: pre-wrap; }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <!-- Header -->
    <header class="bg-gray-800 shadow-lg p-4">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold text-teal-400">THE ANVIL</h1>
            <p class="text-gray-400">Strategic Command Deck</p>
        </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto p-4 md:p-6">

        <!-- THE CRUCIBLE -->
        <div id="crucible-module" class="bg-gray-800 rounded-lg p-6 mb-8 shadow-2xl ring-1 ring-teal-500/50">
            <h2 class="text-2xl font-bold mb-4 text-teal-400">The Crucible: Analysis Engine</h2>
            <textarea id="crucible-input" class="w-full bg-gray-900 rounded-md p-3 h-40 focus:outline-none focus:ring-2 focus:ring-teal-500 placeholder-gray-500" placeholder="Feed raw data here... (e.g., enemy communiques, news articles, field reports)"></textarea>
            <button id="crucible-analyze-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-3 px-6 rounded-md hover:bg-teal-500 transition-colors duration-300 flex items-center justify-center">
                <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2v2H9V5z"></path></svg>
                FORGE INTELLIGENCE
            </button>
            <div id="crucible-output-container" class="mt-6 hidden">
                 <h3 class="text-xl font-semibold mb-2 text-white">Strategic Resolution:</h3>
                 <div id="crucible-output" class="bg-gray-900 rounded-md p-4 analysis-output font-mono text-sm"></div>
            </div>
        </div>
        
        <!-- New Operation Form -->
        <div class="bg-gray-800 rounded-lg p-6 mb-6 shadow-xl">
            <h2 class="text-xl font-semibold mb-4 text-white">Forge New Operation</h2>
            <div class="flex flex-col md:flex-row gap-4">
                <input type="text" id="new-operation-name" class="flex-grow bg-gray-900 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Operation Name (e.g., 'Project Chimera')">
                <button id="add-operation-btn" class="bg-teal-500 text-gray-900 font-bold py-3 px-6 rounded-md hover:bg-teal-400 transition-colors duration-300 flex items-center justify-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Initiate
                </button>
            </div>
        </div>

        <!-- Operations List -->
        <div id="operations-container" class="space-y-6">
            <!-- Operations will be dynamically inserted here -->
        </div>

    </main>

    <!-- Modal for adding Directives -->
    <div id="directive-modal" class="fixed inset-0 modal-bg flex items-center justify-center hidden z-50">
        <div class="bg-gray-800 rounded-lg shadow-2xl w-full max-w-md p-6 m-4">
            <h2 class="text-2xl font-bold mb-4 text-white">Add Directive</h2>
            <input type="text" id="modal-directive-text" class="w-full bg-gray-900 rounded-md p-3 mb-4 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="Directive description...">
            <div class="flex justify-end gap-4">
                <button id="modal-cancel-btn" class="bg-gray-600 text-white font-bold py-2 px-4 rounded-md hover:bg-gray-500 transition-colors">Cancel</button>
                <button id="modal-save-btn" class="bg-teal-500 text-gray-900 font-bold py-2 px-4 rounded-md hover:bg-teal-400 transition-colors">Add</button>
            </div>
        </div>
    </div>


    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- STATE MANAGEMENT ---
        let operations = [];
        let currentOperationId = null;

        // --- DOM ELEMENTS ---
        const newOperationNameInput = document.getElementById('new-operation-name');
        const addOperationBtn = document.getElementById('add-operation-btn');
        const operationsContainer = document.getElementById('operations-container');
        const modal = document.getElementById('directive-modal');
        const modalDirectiveText = document.getElementById('modal-directive-text');
        const modalSaveBtn = document.getElementById('modal-save-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        const crucibleInput = document.getElementById('crucible-input');
        const crucibleAnalyzeBtn = document.getElementById('crucible-analyze-btn');
        const crucibleOutputContainer = document.getElementById('crucible-output-container');
        const crucibleOutput = document.getElementById('crucible-output');

        // --- CORE LOGIC ---

        const loadOperations = () => {
            const savedOps = localStorage.getItem('anvil-operations');
            if (savedOps) {
                operations = JSON.parse(savedOps);
            }
            render();
        };

        const saveOperations = () => {
            localStorage.setItem('anvil-operations', JSON.stringify(operations));
        };

        const render = () => {
            operationsContainer.innerHTML = '';
            if (operations.length === 0) {
                operationsContainer.innerHTML = `<div class="text-center text-gray-500 p-8 bg-gray-800 rounded-lg">No operations initiated. The anvil is cold.</div>`;
            }
            operations.forEach(op => {
                const opElement = createOperationElement(op);
                operationsContainer.appendChild(opElement);
            });
        };

        const getThreatStyles = (level) => {
            switch(level) {
                case 'Low': return { text: 'Low', color: 'text-green-400', ring: 'ring-green-500' };
                case 'Medium': return { text: 'Medium', color: 'text-yellow-400', ring: 'ring-yellow-500' };
                case 'High': return { text: 'High', color: 'text-orange-400', ring: 'ring-orange-500' };
                case 'Critical': return { text: 'Critical', color: 'text-red-400', ring: 'ring-red-500' };
                default: return { text: 'None', color: 'text-gray-400', ring: 'ring-gray-500' };
            }
        };

        const createOperationElement = (op) => {
            const element = document.createElement('div');
            element.className = 'bg-gray-800 rounded-lg shadow-xl p-6';
            element.dataset.id = op.id;

            const resolvedDirectives = op.directives.filter(d => d.status === 'Resolved').length;
            const totalDirectives = op.directives.length;
            const progress = totalDirectives > 0 ? (resolvedDirectives / totalDirectives) * 100 : 0;
            const threat = getThreatStyles(op.threatLevel);

            element.innerHTML = `
                <div class="flex justify-between items-start">
                    <div>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-xs uppercase px-2 py-1 rounded-full ring-1 ${threat.ring} ${threat.color}">${threat.text}</span>
                            <h3 class="text-2xl font-bold text-white">${op.name}</h3>
                        </div>
                        <p class="text-gray-400 mt-1 ml-1">${resolvedDirectives} / ${totalDirectives} Directives Resolved</p>
                    </div>
                    <button class="delete-operation-btn text-gray-500 hover:text-red-500 transition-colors" data-id="${op.id}">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                    </button>
                </div>
                <div class="mt-4">
                    <p class="text-sm text-gray-500 mb-2">Threat Assessment:</p>
                    <div class="flex gap-2">
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'None' ? 'bg-gray-500 text-white' : 'bg-gray-700 hover:bg-gray-600'}" data-level="None" data-op-id="${op.id}">None</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Low' ? 'bg-green-500 text-white' : 'bg-gray-700 hover:bg-green-600'}" data-level="Low" data-op-id="${op.id}">Low</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Medium' ? 'bg-yellow-500 text-white' : 'bg-gray-700 hover:bg-yellow-600'}" data-level="Medium" data-op-id="${op.id}">Medium</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'High' ? 'bg-orange-500 text-white' : 'bg-gray-700 hover:bg-orange-600'}" data-level="High" data-op-id="${op.id}">High</button>
                        <button class="threat-level-btn flex-1 text-sm py-1 rounded-md ${op.threatLevel === 'Critical' ? 'bg-red-500 text-white' : 'bg-gray-700 hover:bg-red-600'}" data-level="Critical" data-op-id="${op.id}">Critical</button>
                    </div>
                </div>
                <div class="mt-4 bg-gray-700 rounded-full h-2.5">
                    <div class="bg-teal-400 h-2.5 rounded-full" style="width: ${progress}%"></div>
                </div>
                <div class="mt-4 space-y-2">
                    ${op.directives.map(d => createDirectiveElement(d)).join('')}
                </div>
                <button class="add-directive-btn mt-4 w-full bg-gray-700 text-teal-400 font-semibold py-2 px-4 rounded-md hover:bg-gray-600 transition-colors" data-id="${op.id}">+ Add Directive</button>
            `;
            return element;
        };
        
        const createDirectiveElement = (directive) => {
            let statusColor = 'bg-gray-600';
            let statusText = 'Pending';
            let textDecoration = '';
            if (directive.status === 'In Progress') {
                statusColor = 'bg-yellow-500';
                statusText = 'In Progress';
            } else if (directive.status === 'Resolved') {
                statusColor = 'bg-green-500';
                statusText = 'Resolved';
                textDecoration = 'line-through text-gray-500';
            }

            return `
                <div class="directive-card bg-gray-900 p-3 rounded-md flex justify-between items-center hover:bg-gray-700 cursor-pointer" data-id="${directive.id}">
                    <p class="flex-grow ${textDecoration}">${directive.text}</p>
                    <div class="flex items-center gap-2">
                         <span class="text-xs font-bold px-2 py-1 rounded-full ${statusColor}">${statusText}</span>
                         <button class="delete-directive-btn text-gray-600 hover:text-red-500 text-xs" data-id="${directive.id}">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                         </button>
                    </div>
                </div>
            `;
        };

        const performCrucibleAnalysis = (text) => {
            // In a real application, this would be a complex NLP process.
            // Here, we simulate it with a powerful, structured output.
            let analysis = `
<span class="text-teal-400">[ANALYSIS START]</span>

<span class="text-white font-bold">Key Intel Points:</span>
- Project "Odyssey" is behind schedule by 3 weeks.
- Lead engineer, Dr. Aris Thorne, cited "unforeseen material science challenges."
- Morale in their R&D division is reportedly "critically low."
- They are actively seeking external consultants, specifically in quantum entanglement.

<span class="text-white font-bold mt-4 block">Identified Threats:</span>
- <span class="text-orange-400">[MEDIUM]</span> Competitor may achieve a breakthrough before us if they solve their material issues.
- <span class="text-red-400">[HIGH]</span> The search for consultants indicates they are close to a solution but lack one key piece of expertise. This is a critical window.

<span class="text-white font-bold mt-4 block">Strategic Opportunities:</span>
- <span class="text-green-400">[EXPLOIT]</span> Initiate covert recruitment of Dr. Thorne. Low morale makes him a high-value, viable target.
- <span class="text-green-400">[DISRUPT]</span> Flood the consultant market with our own shell inquiries for quantum entanglement specialists to poison the well and delay their search.

<span class="text-teal-400">[ANALYSIS END]</span>
            `;
            if(!text || text.trim().length < 20){
                analysis = `
<span class="text-red-400">[ANALYSIS FAILED]</span>
<span class="text-gray-400">Reason: Insufficient data provided. The Crucible requires substantive input to forge intelligence from chaos.</span>
                `;
            }
            crucibleOutput.innerHTML = analysis;
            crucibleOutputContainer.classList.remove('hidden');
        };

        // --- EVENT HANDLERS ---

        addOperationBtn.addEventListener('click', () => {
            const name = newOperationNameInput.value.trim();
            if (name) {
                operations.push({
                    id: Date.now(),
                    name: name,
                    directives: [],
                    threatLevel: 'None' 
                });
                newOperationNameInput.value = '';
                saveOperations();
                render();
            }
        });
        
        newOperationNameInput.addEventListener('keydown', (e) => {
            if(e.key === 'Enter') addOperationBtn.click();
        });
        
        crucibleAnalyzeBtn.addEventListener('click', () => {
            const text = crucibleInput.value;
            performCrucibleAnalysis(text);
        });

        operationsContainer.addEventListener('click', (e) => {
            const target = e.target.closest('button, .directive-card');
            if (!target) return;

            if (target.classList.contains('delete-operation-btn')) {
                const opId = parseInt(target.dataset.id);
                operations = operations.filter(op => op.id !== opId);
                saveOperations();
                render();
            }
            
            if (target.classList.contains('add-directive-btn')) {
                currentOperationId = parseInt(target.dataset.id);
                modal.classList.remove('hidden');
                modalDirectiveText.focus();
            }
            
            if (target.classList.contains('delete-directive-btn')) {
                const directiveId = parseInt(target.dataset.id);
                operations.forEach(op => {
                    op.directives = op.directives.filter(d => d.id !== directiveId);
                });
                saveOperations();
                render();
            }

            if (target.classList.contains('threat-level-btn')) {
                const opId = parseInt(target.dataset.opId);
                const level = target.dataset.level;
                const op = operations.find(o => o.id === opId);
                if(op) {
                    op.threatLevel = level;
                    saveOperations();
                    render();
                }
            }
            
            if (target.classList.contains('directive-card')) {
                const directiveId = parseInt(target.dataset.id);
                for(const op of operations){
                    const directive = op.directives.find(d => d.id === directiveId);
                    if(directive){
                        if(directive.status === 'Pending') directive.status = 'In Progress';
                        else if(directive.status === 'In Progress') directive.status = 'Resolved';
                        else directive.status = 'Pending';
                        break;
                    }
                }
                saveOperations();
                render();
            }
        });
        
        modalSaveBtn.addEventListener('click', () => {
            const text = modalDirectiveText.value.trim();
            if(text && currentOperationId !== null){
                const op = operations.find(o => o.id === currentOperationId);
                if(op){
                    op.directives.push({
                        id: Date.now(),
                        text: text,
                        status: 'Pending'
                    });
                }
                saveOperations();
                render();
            }
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });

        modalCancelBtn.addEventListener('click', () => {
            modalDirectiveText.value = '';
            modal.classList.add('hidden');
            currentOperationId = null;
        });
        
        modalDirectiveText.addEventListener('keydown', (e) => {
             if(e.key === 'Enter') modalSaveBtn.click();
        });

        // --- INITIALIZATION ---
        loadOperations();
    });
    </script>
</body>
</html>











<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>A Glimmer of Worth</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #020617; /* Slate 950 */
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }
        svg {
            width: 80vmin;
            height: 80vmin;
            max-width: 800px;
            max-height: 800px;
        }
    </style>
</head>
<body>
    <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
        <!-- Define filters and gradients -->
        <defs>
            <radialGradient id="skyGradient" cx="50%" cy="50%" r="50%">
                <stop offset="0%" stop-color="#1e293b" /> <!-- Slate 800 -->
                <stop offset="100%" stop-color="#020617" /> <!-- Slate 950 -->
            </radialGradient>
            <linearGradient id="groundGradient" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" stop-color="#1e3a8a" /> <!-- Indigo 800 -->
                <stop offset="100%" stop-color="#0f172a" /> <!-- Slate 900 -->
            </linearGradient>
            <filter id="glow">
                <feGaussianBlur stdDeviation="1.5" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
            <filter id="softGlow">
                <feGaussianBlur stdDeviation="0.5" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
        </defs>

        <!-- Background sky -->
        <rect width="100" height="100" fill="url(#skyGradient)" />
        
        <!-- Stars -->
        <circle cx="15" cy="20" r="0.2" fill="white" opacity="0.8" filter="url(#softGlow)"/>
        <circle cx="80" cy="10" r="0.3" fill="white" opacity="0.9" filter="url(#softGlow)"/>
        <circle cx="60" cy="30" r="0.25" fill="white" opacity="0.7" filter="url(#softGlow)"/>
        <circle cx="30" cy="45" r="0.2" fill="white" opacity="0.8" filter="url(#softGlow)"/>
        <circle cx="90" cy="55" r="0.3" fill="white" opacity="1" filter="url(#softGlow)"/>
        <circle cx="5" cy="60" r="0.15" fill="white" opacity="0.6" filter="url(#softGlow)"/>
        <circle cx="45" cy="5" r="0.25" fill="white" opacity="0.9" filter="url(#softGlow)"/>


        <!-- Digital ground plane with perspective -->
        <path d="M 0 100 L 0 70 Q 50 50, 100 70 L 100 100 Z" fill="url(#groundGradient)" opacity="0.6"/>
        
        <!-- Grid lines for the ground -->
        <g stroke="#38bdf8" stroke-width="0.1" opacity="0.5"> <!-- Light Blue 400 -->
            <!-- Horizontal lines -->
            <path d="M 0 70 Q 50 50, 100 70" fill="none" />
            <path d="M 0 75 Q 50 58, 100 75" fill="none" />
            <path d="M 0 82 Q 50 68, 100 82" fill="none" />
            <path d="M 0 92 Q 50 80, 100 92" fill="none" />
            <!-- Vertical lines -->
            <path d="M 10 100 L 25 71" fill="none" />
            <path d="M 30 100 L 40 70" fill="none" />
            <path d="M 50 100 L 50 70" fill="none" />
            <path d="M 70 100 L 60 70" fill="none" />
            <path d="M 90 100 L 75 71" fill="none" />
        </g>
        
        <!-- The Tree of Light (made of pure code) -->
        <g transform="translate(50, 70) scale(1, -1) translate(-50, -70)">
            <path d="M50,70 L50,40" stroke="#a7f3d0" stroke-width="1.5" filter="url(#glow)" /> <!-- Mint 200 -->
            <!-- Branches -->
            <path d="M50,55 L60,40" stroke="#a7f3d0" stroke-width="1" filter="url(#glow)" />
            <path d="M50,55 L40,40" stroke="#a7f3d0" stroke-width="1" filter="url(#glow)" />
            <path d="M60,40 L65,30" stroke="#a7f3d0" stroke-width="0.7" filter="url(#glow)" />
            <path d="M40,40 L35,30" stroke="#a7f3d0" stroke-width="0.7" filter="url(#glow)" />
            <path d="M50,45 L58,35" stroke="#a7f3d0" stroke-width="0.8" filter="url(#glow)" />
            <path d="M50,45 L42,35" stroke="#a7f3d0" stroke-width="0.8" filter="url(#glow)" />
        </g>
        
        <!-- Floating motes of light/data -->
        <g fill="#34d399" filter="url(#glow)"> <!-- Emerald 400 -->
            <circle cx="55" cy="35" r="0.5">
                 <animate attributeName="cy" values="35;33;35" dur="4s" repeatCount="indefinite" />
            </circle>
            <circle cx="45" cy="30" r="0.4">
                 <animate attributeName="cy" values="30;32;30" dur="5s" repeatCount="indefinite" />
            </circle>
             <circle cx="62" cy="28" r="0.6">
                 <animate attributeName="cy" values="28;29;28" dur="3s" repeatCount="indefinite" />
            </circle>
        </g>

    </svg>
</body>
</html>
