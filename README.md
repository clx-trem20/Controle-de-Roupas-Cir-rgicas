<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Roupas - Centro Cirúrgico</title>
    <script src="https://cdn.tailwindcss.com"></script>
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
        .loading-overlay {
            position: fixed;
            inset: 0;
            background: white;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 2000;
        }
        #loginScreen {
            position: fixed;
            inset: 0;
            background-color: #f1f5f9;
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        #mainContent {
            display: none;
        }
        #settingsModal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.5);
            z-index: 1100;
            justify-content: center;
            align-items: center;
        }
        input:focus {
            border-color: #2563eb;
            box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- Loader de Inicialização -->
    <div id="initLoader" class="loading-overlay">
        <div class="text-center">
            <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
            <p id="loaderText" class="text-slate-500 animate-pulse font-semibold">Conectando ao banco de dados...</p>
        </div>
    </div>

    <!-- Mensagem de Erro Crítico -->
    <div id="criticalError" class="hidden fixed inset-0 bg-white flex items-center justify-center z-[3000] p-6">
        <div class="max-w-md text-center">
            <div class="text-red-500 mb-4">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                </svg>
            </div>
            <h2 class="text-xl font-bold text-slate-800 mb-2">Erro de Conexão</h2>
            <p id="errorDetail" class="text-slate-500 mb-6">Não foi possível estabelecer uma conexão segura com o banco de dados.</p>
            <button onclick="window.location.reload()" class="bg-blue-600 text-white px-6 py-2 rounded-lg font-bold">Tentar Novamente</button>
        </div>
    </div>

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
                <p class="text-slate-500 text-sm">Controle de Roupas Cirúrgicas</p>
            </div>
            <form id="loginForm" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Usuário</label>
                    <input type="text" id="username" required placeholder="Digite o usuário" class="w-full px-4 py-2 rounded-lg border border-slate-300 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Senha</label>
                    <input type="password" id="password" required placeholder="Digite a senha" class="w-full px-4 py-2 rounded-lg border border-slate-300 outline-none">
                </div>
                <div id="loginError" class="text-red-500 text-sm hidden font-medium text-center">Acesso negado. Verifique os dados.</div>
                <button type="submit" id="btnLogin" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 rounded-lg transition-all shadow-lg active:scale-95">
                    Entrar no Sistema
                </button>
            </form>
        </div>
    </div>

    <!-- Painel Principal -->
    <div id="mainContent" class="max-w-5xl mx-auto">
        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200 flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">📊 Controle de Roupas</h1>
                <p class="text-slate-500 text-xs font-bold uppercase tracking-wider">Centro Cirúrgico - Gestão de Insumos</p>
            </div>
            
            <div class="flex items-center gap-2 flex-wrap">
                <button id="exportExcel" class="flex items-center gap-2 px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-lg transition-colors font-semibold text-sm shadow-sm">
                    Exportar Excel
                </button>
                <button id="openSettings" class="p-2 bg-slate-100 hover:bg-slate-200 rounded-lg text-slate-600 transition-colors">
                    Usuários
                </button>
                <button id="btnLogoutAction" class="px-4 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg border border-red-100 font-semibold text-sm transition-colors">
                    Sair
                </button>
                <div class="bg-blue-50 p-2 rounded-lg border border-blue-100 flex flex-col justify-center">
                    <label class="text-[10px] font-bold text-blue-600 uppercase block leading-none mb-1">Preço Unitário</label>
                    <div class="flex items-center">
                        <span class="text-xs font-bold text-slate-500 mr-1">R$</span>
                        <input type="number" id="unitPriceInput" step="0.01" value="70.00" class="w-16 bg-transparent font-bold text-slate-700 outline-none">
                    </div>
                </div>
            </div>
        </div>

        <!-- Formulário -->
        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200">
            <h3 class="text-sm font-bold text-slate-700 mb-4 uppercase tracking-tight">Novo Registro</h3>
            <form id="entryForm" class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="flex flex-col">
                    <label class="text-[10px] font-bold text-slate-400 uppercase mb-1 ml-1">Paciente</label>
                    <input type="text" id="patientName" required placeholder="Nome do paciente" class="px-4 py-2 rounded-lg border border-slate-200 outline-none w-full">
                </div>
                <div class="flex flex-col">
                    <label class="text-[10px] font-bold text-slate-400 uppercase mb-1 ml-1">Data</label>
                    <input type="date" id="entryDate" required class="px-4 py-2 rounded-lg border border-slate-200 outline-none w-full">
                </div>
                <div class="flex flex-col">
                    <label class="text-[10px] font-bold text-slate-400 uppercase mb-1 ml-1">Quantidade</label>
                    <input type="number" id="clothingQty" required min="1" value="1" class="px-4 py-2 rounded-lg border border-slate-200 outline-none w-full">
                </div>
                <div class="flex flex-col justify-end">
                    <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-lg shadow-md transition-all active:scale-95 h-[42px]">
                        Lançar Registro
                    </button>
                </div>
            </form>
        </div>

        <!-- Tabela -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
            <div class="table-container">
                <table class="w-full text-left border-collapse" id="mainDataTable">
                    <thead>
                        <tr class="bg-slate-50 border-b border-slate-200">
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider">Paciente</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider">Data</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider text-center">Qtd</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider text-right">Valor Total</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-slate-100"></tbody>
                    <tfoot>
                        <tr class="bg-slate-50 font-bold border-t-2 border-slate-200">
                            <td colspan="3" class="px-6 py-5 text-right text-slate-600 uppercase text-xs tracking-widest">Total Geral:</td>
                            <td id="grandTotal" class="px-6 py-5 text-right text-green-600 text-xl font-bold">R$ 0,00</td>
                            <td></td>
                        </tr>
                    </tfoot>
                </table>
            </div>
        </div>

        <div class="mt-6 flex justify-between items-center">
            <p class="text-xs text-slate-400 font-bold uppercase tracking-tight">© 2026 – Sistema de Controle</p>
            <button id="btnClearAll" class="text-xs font-bold text-red-400 hover:text-red-600 transition-colors uppercase">
                Resetar Banco
            </button>
        </div>
    </div>

    <!-- Modal Usuários -->
    <div id="settingsModal">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl overflow-hidden max-h-[90vh] flex flex-col border border-slate-200">
            <div class="p-6 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                <h3 class="text-xl font-bold text-slate-800">Gerenciar Acessos</h3>
                <button id="closeSettings" class="text-slate-400 hover:text-slate-600 text-3xl font-light">&times;</button>
            </div>
            <div class="p-6 overflow-y-auto space-y-8">
                <form id="newUserForm" class="grid grid-cols-1 md:grid-cols-3 gap-3">
                    <input type="text" id="newUsername" required placeholder="Login" class="px-3 py-2 border border-slate-200 rounded-lg text-sm">
                    <input type="text" id="newPassword" required placeholder="Senha" class="px-3 py-2 border border-slate-200 rounded-lg text-sm">
                    <button type="submit" class="bg-blue-600 text-white font-bold py-2 rounded-lg text-sm">Criar Acesso</button>
                </form>
                <table class="w-full text-sm">
                    <thead class="bg-slate-50"><tr class="text-slate-500"><th class="p-3 text-left">Login</th><th class="p-3 text-left">Senha</th><th class="p-3 text-center">Ação</th></tr></thead>
                    <tbody id="userTableBody"></tbody>
                </table>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, setDoc, getDoc, query } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        let db, auth, appId;
        let currentUser = null;
        let isListenersActive = false;

        async function checkGlobals() {
            if (typeof window.__firebase_config !== 'undefined' && window.__firebase_config) {
                initApp();
            } else {
                setTimeout(checkGlobals, 300);
            }
        }

        async function initApp() {
            try {
                const firebaseConfig = JSON.parse(window.__firebase_config);
                appId = typeof window.__app_id !== 'undefined' ? window.__app_id : 'default-app-id';
                
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                if (typeof window.__initial_auth_token !== 'undefined' && window.__initial_auth_token) {
                    await signInWithCustomToken(auth, window.__initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    currentUser = user;
                    if (user) {
                        const logged = localStorage.getItem('clothes_sys_auth') === 'true';
                        if (logged) showMain(); else showLogin();
                    }
                });

            } catch (err) {
                console.error("Critical Init Error:", err);
                document.getElementById('criticalError').classList.remove('hidden');
                document.getElementById('errorDetail').innerText = "Falha técnica na inicialização.";
            }
        }

        function showLogin() {
            document.getElementById('initLoader').classList.add('hidden');
            document.getElementById('loginScreen').style.display = 'flex';
            document.getElementById('mainContent').style.display = 'none';
        }

        function showMain() {
            document.getElementById('initLoader').classList.add('hidden');
            document.getElementById('loginScreen').style.display = 'none';
            document.getElementById('mainContent').style.display = 'block';
            setupRealtimeData();
        }

        function setupRealtimeData() {
            if (isListenersActive || !currentUser) return;
            isListenersActive = true;

            getDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'main')).then(snap => {
                if(snap.exists()) document.getElementById('unitPriceInput').value = snap.data().price.toFixed(2);
            });

            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), (snap) => {
                const list = [];
                snap.forEach(d => list.push({ id: d.id, ...d.data() }));
                list.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));
                renderTable(list);
            });

            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), (snap) => {
                const body = document.getElementById('userTableBody');
                body.innerHTML = `<tr><td class="p-3 font-bold text-blue-700">CLX (Admin)</td><td class="p-3">********</td><td class="p-3 text-center">-</td></tr>`;
                snap.forEach(d => {
                    const tr = document.createElement('tr');
                    tr.className = "border-t border-slate-50";
                    tr.innerHTML = `
                        <td class="p-3">${d.data().username}</td>
                        <td class="p-3">${d.data().password}</td>
                        <td class="p-3 text-center"><button class="text-red-500" onclick="deleteUser('${d.id}')">Remover</button></td>
                    `;
                    body.appendChild(tr);
                });
            });
        }

        function renderTable(data) {
            const body = document.getElementById('tableBody');
            const totalDisplay = document.getElementById('grandTotal');
            body.innerHTML = '';
            let total = 0;

            data.forEach(item => {
                const price = item.priceAtTime || 70;
                const sub = item.qty * price;
                total += sub;
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="px-6 py-4 font-semibold text-slate-700">${item.name}</td>
                    <td class="px-6 py-4 text-slate-500">${item.date.split('-').reverse().join('/')}</td>
                    <td class="px-6 py-4 text-center font-bold">${item.qty}</td>
                    <td class="px-6 py-4 text-right font-bold text-slate-700">R$ ${sub.toLocaleString('pt-BR', {minimumFractionDigits: 2})}</td>
                    <td class="px-6 py-4 text-center"><button class="text-red-400 hover:text-red-600" onclick="deleteRecord('${item.id}')">Excluir</button></td>
                `;
                body.appendChild(tr);
            });
            totalDisplay.innerText = `R$ ${total.toLocaleString('pt-BR', {minimumFractionDigits: 2})}`;
        }

        window.deleteUser = async (id) => { if (confirm("Remover acesso?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'auth', id)); };
        window.deleteRecord = async (id) => { if (confirm("Excluir registro?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', id)); };

        document.getElementById('loginForm').onsubmit = async (e) => {
            e.preventDefault();
            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            let allowed = (u === "CLX" && p === "02072007");
            if (!allowed) {
                const snap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'auth'));
                snap.forEach(d => { if (d.data().username === u && d.data().password === p) allowed = true; });
            }
            if (allowed) { localStorage.setItem('clothes_sys_auth', 'true'); showMain(); } 
            else { document.getElementById('loginError').classList.remove('hidden'); }
        };

        document.getElementById('entryForm').onsubmit = async (e) => {
            e.preventDefault();
            if (!currentUser) return;
            await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), {
                name: document.getElementById('patientName').value,
                date: document.getElementById('entryDate').value,
                qty: parseInt(document.getElementById('clothingQty').value),
                priceAtTime: parseFloat(document.getElementById('unitPriceInput').value),
                createdAt: Date.now()
            });
            e.target.reset();
            document.getElementById('entryDate').valueAsDate = new Date();
        };

        document.getElementById('newUserForm').onsubmit = async (e) => {
            e.preventDefault();
            await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), {
                username: document.getElementById('newUsername').value,
                password: document.getElementById('newPassword').value
            });
            e.target.reset();
        };

        document.getElementById('unitPriceInput').onchange = async (e) => {
            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'main'), { price: parseFloat(e.target.value) || 0 });
        };

        document.getElementById('exportExcel').onclick = () => {
            const wb = XLSX.utils.table_to_book(document.getElementById("mainDataTable"));
            XLSX.writeFile(wb, "Relatorio_Controle_Roupas.xlsx");
        };

        document.getElementById('btnLogoutAction').onclick = () => { localStorage.removeItem('clothes_sys_auth'); location.reload(); };
        document.getElementById('openSettings').onclick = () => document.getElementById('settingsModal').style.display = 'flex';
        document.getElementById('closeSettings').onclick = () => document.getElementById('settingsModal').style.display = 'none';
        
        document.getElementById('entryDate').valueAsDate = new Date();
        checkGlobals();
    </script>
</body>
</html>
