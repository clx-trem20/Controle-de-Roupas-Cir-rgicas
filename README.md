<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Roupas - Centro Cirúrgico</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Biblioteca para exportação Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }
        .table-container {
            overflow-x: auto;
        }
        input::placeholder {
            color: #cbd5e1;
        }
        .loading-overlay {
            position: fixed;
            inset: 0;
            background: rgba(255,255,255,0.7);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 100;
        }
        #loginScreen {
            position: fixed;
            inset: 0;
            background-color: #f1f5f9;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 200;
        }
        #mainContent {
            display: none;
        }
        #settingsModal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.5);
            z-index: 300;
            justify-content: center;
            align-items: center;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- Tela de Login -->
    <div id="loginScreen">
        <div class="bg-white p-8 rounded-2xl shadow-xl border border-slate-200 w-full max-w-md">
            <div class="text-center mb-8">
                <div class="bg-blue-100 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                    </svg>
                </div>
                <h2 class="text-2xl font-bold text-slate-800">Acesso Restrito</h2>
                <p class="text-slate-500 text-sm">Entre com suas credenciais</p>
            </div>
            <form id="loginForm" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Usuário</label>
                    <input type="text" id="username" required placeholder="Digite o usuário" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Senha</label>
                    <input type="password" id="password" required placeholder="Digite a senha" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>
                <div id="loginError" class="text-red-500 text-sm hidden font-medium">Usuário ou senha incorretos.</div>
                <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 rounded-lg transition-all shadow-lg active:scale-95">
                    Entrar no Sistema
                </button>
            </form>
        </div>
    </div>

    <!-- Modal de Configurações -->
    <div id="settingsModal">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl overflow-hidden max-h-[90vh] flex flex-col">
            <div class="p-6 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                <h3 class="text-xl font-bold text-slate-800 flex items-center gap-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-slate-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                    </svg>
                    Configurações do Sistema
                </h3>
                <button id="closeSettings" class="text-slate-400 hover:text-slate-600">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                </button>
            </div>
            <div class="p-6 overflow-y-auto space-y-8">
                <div>
                    <h4 class="font-bold text-slate-700 mb-4 flex items-center gap-2">
                        <span class="w-2 h-2 bg-blue-500 rounded-full"></span>
                        Cadastrar Novo Acesso
                    </h4>
                    <form id="newUserForm" class="grid grid-cols-1 md:grid-cols-3 gap-4 bg-slate-50 p-4 rounded-xl border border-slate-200">
                        <div>
                            <label class="block text-xs font-bold text-slate-500 uppercase mb-1">Usuário</label>
                            <input type="text" id="newUsername" required placeholder="Ex: recepcao" 
                                class="w-full px-3 py-2 rounded-lg border border-slate-300 outline-none focus:ring-2 focus:ring-blue-500">
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-slate-500 uppercase mb-1">Senha</label>
                            <input type="text" id="newPassword" required placeholder="Senha forte" 
                                class="w-full px-3 py-2 rounded-lg border border-slate-300 outline-none focus:ring-2 focus:ring-blue-500">
                        </div>
                        <div class="flex items-end">
                            <button type="submit" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg hover:bg-blue-700 transition-colors">
                                Salvar Acesso
                            </button>
                        </div>
                    </form>
                </div>

                <div>
                    <h4 class="font-bold text-slate-700 mb-4 flex items-center gap-2">
                        <span class="w-2 h-2 bg-green-500 rounded-full"></span>
                        Usuários Cadastrados
                    </h4>
                    <div class="border border-slate-200 rounded-xl overflow-hidden">
                        <table class="w-full text-sm">
                            <thead class="bg-slate-50 border-b border-slate-200 text-slate-500 font-bold uppercase text-[10px] tracking-wider">
                                <tr>
                                    <th class="px-4 py-2 text-left">Login</th>
                                    <th class="px-4 py-2 text-left">Senha</th>
                                    <th class="px-4 py-2 text-center">Ações</th>
                                </tr>
                            </thead>
                            <tbody id="userTableBody"></tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Overlay de Carregamento -->
    <div id="loader" class="loading-overlay">
        <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
    </div>

    <!-- Conteúdo Principal -->
    <div id="mainContent" class="max-w-5xl mx-auto">
        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200 flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div class="flex items-center gap-4">
                <div>
                    <h1 class="text-2xl font-bold text-slate-800 mb-1">📊 Controle de Roupas Cirúrgicas</h1>
                    <div class="flex items-center gap-2">
                        <p class="text-slate-500 text-xs">Dados em nuvem</p>
                        <span class="text-[10px] bg-green-100 text-green-700 px-2 py-0.5 rounded-full font-bold uppercase tracking-wider">Sessão Ativa</span>
                    </div>
                </div>
            </div>
            
            <div class="flex items-center gap-3">
                <!-- Botão Excel -->
                <button id="exportExcel" class="flex items-center gap-2 px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-lg transition-colors shadow-sm font-semibold text-sm">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                    </svg>
                    Exportar Excel
                </button>

                <!-- Botão Sair -->
                <button onclick="logout()" class="flex items-center gap-2 px-4 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg transition-colors border border-red-100 font-semibold text-sm">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
                    </svg>
                    Sair
                </button>
                
                <button id="openSettings" class="p-2 bg-slate-100 hover:bg-slate-200 rounded-lg transition-colors text-slate-600">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                    </svg>
                </button>
                <div class="bg-blue-50 p-2 md:p-3 rounded-lg border border-blue-100 flex flex-col">
                    <label class="text-[10px] font-bold text-blue-600 uppercase">Preço Un.</label>
                    <input type="number" id="unitPriceInput" step="0.01" value="70.00" 
                        class="w-20 bg-transparent focus:outline-none font-bold text-slate-700">
                </div>
            </div>
        </div>

        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200">
            <h2 class="text-lg font-semibold text-slate-700 mb-4">Nova Entrada</h2>
            <form id="entryForm" class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Nome da Paciente</label>
                    <input type="text" id="patientName" required placeholder="Ex: Maria Oliveira" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none transition-all">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Data</label>
                    <input type="date" id="entryDate" required 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none transition-all">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Qtd. de Roupas</label>
                    <input type="number" id="clothingQty" required min="1" value="1" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none transition-all">
                </div>
                <div class="flex items-end">
                    <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg transition-colors shadow-md">
                        Adicionar Registro
                    </button>
                </div>
            </form>
        </div>

        <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
            <div class="p-4 bg-slate-50 border-b border-slate-200 flex justify-between items-center">
                <h2 class="text-lg font-semibold text-slate-700">Histórico de Cobrança</h2>
                <span class="text-[10px] text-slate-400 font-bold uppercase tracking-widest">Painel Administrativo</span>
            </div>
            
            <div class="table-container">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50">
                            <th class="px-6 py-3 text-sm font-semibold text-slate-600 border-b">Paciente</th>
                            <th class="px-6 py-3 text-sm font-semibold text-slate-600 border-b">Data</th>
                            <th class="px-6 py-3 text-sm font-semibold text-slate-600 border-b text-center">Qtd</th>
                            <th class="px-6 py-3 text-sm font-semibold text-slate-600 border-b text-right">Valor Total</th>
                            <th class="px-6 py-3 text-sm font-semibold text-slate-600 border-b text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody"></tbody>
                    <tfoot>
                        <tr class="bg-slate-50 font-bold">
                            <td colspan="3" class="px-6 py-4 text-right text-slate-700">TOTAL GERAL:</td>
                            <td id="grandTotal" class="px-6 py-4 text-right text-green-600 text-lg">R$ 0,00</td>
                            <td></td>
                        </tr>
                    </tfoot>
                </table>
            </div>
        </div>

        <div class="mt-4 flex justify-end">
            <button id="btnClearAll" class="text-sm text-red-500 hover:text-red-700 underline px-2 py-1">
                Limpar todos os dados
            </button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, writeBatch, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // Configuração e Inicialização
        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        // Elementos DOM
        const loginScreen = document.getElementById('loginScreen');
        const mainContent = document.getElementById('mainContent');
        const loginForm = document.getElementById('loginForm');
        const loginError = document.getElementById('loginError');
        const form = document.getElementById('entryForm');
        const tableBody = document.getElementById('tableBody');
        const grandTotalDisplay = document.getElementById('grandTotal');
        const unitPriceInput = document.getElementById('unitPriceInput');
        const btnClearAll = document.getElementById('btnClearAll');
        const loader = document.getElementById('loader');
        const settingsModal = document.getElementById('settingsModal');
        const openSettings = document.getElementById('openSettings');
        const closeSettings = document.getElementById('closeSettings');
        const newUserForm = document.getElementById('newUserForm');
        const userTableBody = document.getElementById('userTableBody');
        const exportExcelBtn = document.getElementById('exportExcel');

        let userState = null;
        let authInitialized = false;
        let currentRecords = [];

        // --- REGRA DE OURO: Autenticação ANTES de tudo ---
        async function performLogin() {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (err) {
                console.error("Erro na autenticação:", err);
            }
        }

        onAuthStateChanged(auth, (user) => {
            userState = user;
            if (!authInitialized) {
                authInitialized = true;
                if (sessionStorage.getItem('isLogged') === 'true') {
                    showApp();
                }
            }
        });

        async function ensureAuth() {
            if (userState) return userState;
            await performLogin();
            return new Promise((resolve) => {
                const unsubscribe = onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userState = user;
                        unsubscribe();
                        resolve(user);
                    }
                });
            });
        }

        // --- Lógica de Login ---
        loginForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const userIn = document.getElementById('username').value;
            const passIn = document.getElementById('password').value;

            loader.style.display = 'flex';
            const user = await ensureAuth();
            if (!user) {
                loader.style.display = 'none';
                return;
            }

            try {
                const coll = collection(db, 'artifacts', appId, 'public', 'data', 'auth');
                const snap = await getDocs(coll);
                const validUsers = [];
                snap.forEach(d => validUsers.push(d.data()));
                
                if (!validUsers.some(u => u.username === "CLX")) {
                    validUsers.push({ username: "CLX", password: "02072007" });
                }

                const isValid = validUsers.some(u => u.username === userIn && u.password === passIn);
                loader.style.display = 'none';

                if (isValid) {
                    sessionStorage.setItem('isLogged', 'true');
                    showApp();
                } else {
                    loginError.classList.remove('hidden');
                }
            } catch (err) {
                console.error("Erro validar login:", err);
                loader.style.display = 'none';
            }
        });

        function showApp() {
            loginScreen.style.display = 'none';
            mainContent.style.display = 'block';
            startAppLogic();
        }

        window.logout = () => {
            if(confirm('Deseja realmente sair?')) {
                sessionStorage.removeItem('isLogged');
                window.location.reload();
            }
        };

        async function startAppLogic() {
            const user = await ensureAuth();
            if (!user) return;
            loadConfig();
            setupRegistrosListener();
            setupUsersListener();
        }

        async function loadConfig() {
            const user = await ensureAuth();
            if (!user) return;
            try {
                const configDoc = doc(db, 'artifacts', appId, 'public', 'data', 'config', 'price');
                const snap = await getDoc(configDoc);
                if (snap.exists()) {
                    unitPriceInput.value = snap.data().value.toFixed(2);
                }
            } catch (e) { console.error("Erro config:", e); }
        }

        unitPriceInput.addEventListener('change', async (e) => {
            const user = await ensureAuth();
            if (!user) return;
            try {
                const newPrice = parseFloat(e.target.value) || 0;
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'price'), { value: newPrice });
            } catch (e) { console.error("Erro update price:", e); }
        });

        function setupRegistrosListener() {
            const collRef = collection(db, 'artifacts', appId, 'public', 'data', 'registros');
            onSnapshot(collRef, (snapshot) => {
                const records = [];
                snapshot.forEach(doc => records.push({ id: doc.id, ...doc.data() }));
                records.sort((a, b) => new Date(b.date) - new Date(a.date));
                currentRecords = records;
                renderTable(records);
            }, (err) => console.error("Erro registros listener:", err));
        }

        function renderTable(records) {
            tableBody.innerHTML = '';
            let totalGeneral = 0;
            records.forEach((record) => {
                const rowTotal = record.qty * (record.unitPriceAtTime || 70);
                totalGeneral += rowTotal;
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 border-b border-slate-100";
                tr.innerHTML = `
                    <td class="px-6 py-4 text-slate-800 font-medium">${record.name}</td>
                    <td class="px-6 py-4 text-slate-600">${formatDate(record.date)}</td>
                    <td class="px-6 py-4 text-center text-slate-600">${record.qty}</td>
                    <td class="px-6 py-4 text-right text-slate-800 font-semibold">R$ ${rowTotal.toFixed(2).replace('.', ',')}</td>
                    <td class="px-6 py-4 text-center">
                        <button class="delete-btn text-red-400 hover:text-red-600" data-id="${record.id}">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                            </svg>
                        </button>
                    </td>
                `;
                tableBody.appendChild(tr);
            });
            document.querySelectorAll('.delete-btn').forEach(btn => {
                btn.onclick = async (e) => {
                    const id = e.currentTarget.getAttribute('data-id');
                    if(confirm('Excluir este registro?')) {
                        const user = await ensureAuth();
                        if(user) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', id));
                    }
                };
            });
            grandTotalDisplay.innerText = `R$ ${totalGeneral.toFixed(2).replace('.', ',')}`;
        }

        // Função Exportar Excel
        exportExcelBtn.onclick = () => {
            if (currentRecords.length === 0) {
                alert("Não há dados para exportar.");
                return;
            }

            // Mapear dados para o formato Excel
            const excelData = currentRecords.map(r => ({
                "Paciente": r.name,
                "Data": formatDate(r.date),
                "Quantidade": r.qty,
                "Valor Unit. (R$)": (r.unitPriceAtTime || 70).toFixed(2),
                "Total (R$)": (r.qty * (r.unitPriceAtTime || 70)).toFixed(2)
            }));

            const worksheet = XLSX.utils.json_to_sheet(excelData);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "Controle de Roupas");

            // Gerar nome do arquivo com data atual
            const fileName = `Controle_Roupas_${new Date().toISOString().split('T')[0]}.xlsx`;
            XLSX.writeFile(workbook, fileName);
        };

        function setupUsersListener() {
            const coll = collection(db, 'artifacts', appId, 'public', 'data', 'auth');
            onSnapshot(coll, (snap) => {
                userTableBody.innerHTML = '';
                const masterTr = document.createElement('tr');
                masterTr.innerHTML = `<td class="px-4 py-2 font-bold">CLX (Master)</td><td class="px-4 py-2">********</td><td class="px-4 py-2 text-center text-slate-400">Protegido</td>`;
                userTableBody.appendChild(masterTr);
                
                snap.forEach(docSnap => {
                    const data = docSnap.data();
                    const tr = document.createElement('tr');
                    tr.className = "border-t border-slate-100";
                    tr.innerHTML = `
                        <td class="px-4 py-2 text-slate-700">${data.username}</td>
                        <td class="px-4 py-2 text-slate-700">${data.password}</td>
                        <td class="px-4 py-2 text-center">
                            <button class="del-user text-red-400 hover:text-red-600" data-id="${docSnap.id}">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                </svg>
                            </button>
                        </td>
                    `;
                    userTableBody.appendChild(tr);
                });
                document.querySelectorAll('.del-user').forEach(btn => {
                    btn.onclick = async (e) => {
                        const id = e.currentTarget.getAttribute('data-id');
                        if(confirm('Excluir acesso?')) {
                            const user = await ensureAuth();
                            if(user) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'auth', id));
                        }
                    }
                });
            }, (err) => console.error("Erro users listener:", err));
        }

        newUserForm.onsubmit = async (e) => {
            e.preventDefault();
            const user = await ensureAuth();
            if (!user) return;
            const u = document.getElementById('newUsername').value;
            const p = document.getElementById('newPassword').value;
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), { username: u, password: p });
                newUserForm.reset();
            } catch(e) { console.error("Erro add user:", e); }
        };

        form.onsubmit = async (e) => {
            e.preventDefault();
            const user = await ensureAuth();
            if (!user) return;
            loader.style.display = 'flex';
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), {
                    name: document.getElementById('patientName').value,
                    date: document.getElementById('entryDate').value,
                    qty: parseInt(document.getElementById('clothingQty').value),
                    unitPriceAtTime: parseFloat(unitPriceInput.value),
                    createdAt: new Date().toISOString()
                });
                form.reset();
                document.getElementById('entryDate').valueAsDate = new Date();
            } catch (err) { console.error("Erro add registro:", err); }
            finally { loader.style.display = 'none'; }
        };

        btnClearAll.onclick = async () => {
            const user = await ensureAuth();
            if(!user || !confirm('Apagar TUDO?')) return;
            loader.style.display = 'flex';
            try {
                const qSnap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'registros'));
                const batch = writeBatch(db);
                qSnap.forEach(d => batch.delete(d.ref));
                await batch.commit();
            } catch(e) { console.error("Erro clear all:", e); }
            finally { loader.style.display = 'none'; }
        };

        openSettings.onclick = () => settingsModal.style.display = 'flex';
        closeSettings.onclick = () => settingsModal.style.display = 'none';

        function formatDate(dateStr) {
            if(!dateStr) return "-";
            const [year, month, day] = dateStr.split('-');
            return `${day}/${month}/${year}`;
        }

        window.onload = async () => {
            document.getElementById('entryDate').valueAsDate = new Date();
            await performLogin();
        };
    </script>
</body>
</html>
