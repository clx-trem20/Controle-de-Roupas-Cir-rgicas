<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Roupas - Centro Cirúrgico</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        
        :root {
            --primary: #2563eb;
            --primary-hover: #1d4ed8;
            --bg: #f8fafc;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--bg);
            color: #1e293b;
        }

        .loading-overlay {
            position: fixed;
            inset: 0;
            background: #ffffff;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 9999;
            transition: opacity 0.4s ease-out;
        }

        .table-container::-webkit-scrollbar {
            height: 6px;
        }
        .table-container::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 10px;
        }

        #loginScreen, #mainContent {
            display: none;
        }

        input:focus {
            ring: 2px;
            ring-color: var(--primary);
            border-color: var(--primary);
        }

        .btn-primary {
            background-color: var(--primary);
            transition: all 0.2s;
        }

        .btn-primary:hover {
            background-color: var(--primary-hover);
            transform: translateY(-1px);
        }

        #settingsModal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(15, 23, 42, 0.6);
            backdrop-filter: blur(4px);
            z-index: 2000;
            justify-content: center;
            align-items: center;
            padding: 1rem;
        }
    </style>
</head>
<body class="antialiased min-h-screen">

    <!-- Loader Inicial -->
    <div id="initLoader" class="loading-overlay">
        <div class="relative">
            <div class="w-16 h-16 border-4 border-blue-100 border-t-blue-600 rounded-full animate-spin"></div>
        </div>
        <p class="mt-4 text-slate-600 font-medium animate-pulse">A carregar sistema...</p>
    </div>

    <!-- Tela de Login -->
    <div id="loginScreen" class="min-h-screen w-full flex items-center justify-center p-4 bg-slate-50">
        <div class="bg-white p-8 rounded-3xl shadow-2xl border border-slate-200 w-full max-w-md relative overflow-hidden">
            <div class="absolute top-0 left-0 w-full h-2 bg-blue-600"></div>
            
            <div class="text-center mb-10">
                <div class="bg-blue-50 w-20 h-20 rounded-2xl flex items-center justify-center mx-auto mb-6 rotate-3 shadow-inner">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10 text-blue-600 -rotate-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                    </svg>
                </div>
                <h2 class="text-3xl font-extrabold text-slate-900 tracking-tight">Bem-vindo</h2>
                <p class="text-slate-500 mt-2 font-medium">Controle de Roupas Cirúrgicas</p>
            </div>

            <form id="loginForm" class="space-y-5">
                <div>
                    <label class="block text-sm font-bold text-slate-700 mb-2 ml-1">Usuário</label>
                    <input type="text" id="username" required placeholder="Digite o seu utilizador" 
                           class="w-full px-5 py-4 rounded-2xl border border-slate-200 bg-slate-50 outline-none focus:bg-white focus:ring-4 focus:ring-blue-50 transition-all">
                </div>
                <div>
                    <label class="block text-sm font-bold text-slate-700 mb-2 ml-1">Senha</label>
                    <input type="password" id="password" required placeholder="••••••••" 
                           class="w-full px-5 py-4 rounded-2xl border border-slate-200 bg-slate-50 outline-none focus:bg-white focus:ring-4 focus:ring-blue-50 transition-all">
                </div>
                <div id="loginError" class="text-red-500 text-sm hidden font-semibold text-center bg-red-50 py-2 rounded-lg border border-red-100">
                    Acesso negado. Credenciais inválidas.
                </div>
                <button type="submit" class="w-full btn-primary text-white font-bold py-4 rounded-2xl shadow-lg shadow-blue-200 flex items-center justify-center gap-2">
                    Entrar no Painel
                </button>
            </form>
        </div>
    </div>

    <!-- Conteúdo Principal -->
    <div id="mainContent" class="max-w-6xl mx-auto py-8 px-4">
        
        <!-- Header -->
        <header class="mb-10 flex flex-col md:flex-row md:items-end justify-between gap-6">
            <div class="space-y-1">
                <div class="flex items-center gap-3">
                    <span class="bg-blue-600 text-white p-2 rounded-xl shadow-lg">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
                        </svg>
                    </span>
                    <h1 class="text-3xl font-black text-slate-800 tracking-tight">Controle de Roupas</h1>
                </div>
                <p class="text-slate-500 font-medium ml-12">Monitorização de Insumos - Centro Cirúrgico</p>
            </div>

            <div class="flex items-center gap-3 flex-wrap">
                <button id="exportExcel" class="flex items-center gap-2 px-5 py-3 bg-emerald-600 hover:bg-emerald-700 text-white rounded-2xl font-bold text-sm shadow-md transition-all">
                    Excel
                </button>
                <button id="openSettings" class="p-3 bg-white border border-slate-200 text-slate-600 rounded-2xl hover:bg-slate-50 transition-all shadow-sm">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" />
                    </svg>
                </button>
                <button id="btnLogoutAction" class="px-5 py-3 bg-rose-50 text-rose-600 hover:bg-rose-100 rounded-2xl font-bold text-sm border border-rose-100 transition-all">
                    Sair
                </button>
                
                <div class="bg-white px-4 py-2 rounded-2xl border border-slate-200 shadow-sm flex flex-col justify-center">
                    <span class="text-[10px] font-black text-slate-400 uppercase tracking-tighter">Preço Unitário</span>
                    <div class="flex items-center font-bold text-slate-700">
                        <span class="text-xs mr-1 text-slate-400">R$</span>
                        <input type="number" id="unitPriceInput" step="0.01" value="70.00" class="w-16 bg-transparent outline-none">
                    </div>
                </div>
            </div>
        </header>

        <!-- Formulário de Entrada -->
        <div class="bg-white rounded-3xl shadow-sm border border-slate-200 p-8 mb-8">
            <h3 class="text-lg font-bold text-slate-800 mb-6 flex items-center gap-2">
                <span class="w-2 h-6 bg-blue-600 rounded-full"></span>
                Novo Lançamento
            </h3>
            <form id="entryForm" class="grid grid-cols-1 md:grid-cols-4 gap-6">
                <div class="space-y-2">
                    <label class="text-xs font-bold text-slate-500 uppercase ml-1">Paciente</label>
                    <input type="text" id="patientName" required placeholder="Nome completo" class="w-full px-5 py-3 rounded-2xl border border-slate-200 bg-slate-50 focus:bg-white outline-none">
                </div>
                <div class="space-y-2">
                    <label class="text-xs font-bold text-slate-500 uppercase ml-1">Data</label>
                    <input type="date" id="entryDate" required class="w-full px-5 py-3 rounded-2xl border border-slate-200 bg-slate-50 focus:bg-white outline-none">
                </div>
                <div class="space-y-2">
                    <label class="text-xs font-bold text-slate-500 uppercase ml-1">Quantidade</label>
                    <input type="number" id="clothingQty" required min="1" value="1" class="w-full px-5 py-3 rounded-2xl border border-slate-200 bg-slate-50 focus:bg-white outline-none font-bold">
                </div>
                <div class="flex items-end">
                    <button type="submit" class="w-full btn-primary text-white font-bold py-3.5 rounded-2xl shadow-lg shadow-blue-100">
                        Registar
                    </button>
                </div>
            </form>
        </div>

        <!-- Tabela -->
        <div class="bg-white rounded-3xl shadow-sm border border-slate-200 overflow-hidden">
            <div class="table-container overflow-x-auto">
                <table class="w-full text-left" id="mainDataTable">
                    <thead>
                        <tr class="bg-slate-50/50 border-b border-slate-100">
                            <th class="px-8 py-5 text-xs font-bold text-slate-400 uppercase tracking-widest">Paciente</th>
                            <th class="px-8 py-5 text-xs font-bold text-slate-400 uppercase tracking-widest">Data</th>
                            <th class="px-8 py-5 text-xs font-bold text-slate-400 uppercase tracking-widest text-center">Qtd</th>
                            <th class="px-8 py-5 text-xs font-bold text-slate-400 uppercase tracking-widest text-right">Subtotal</th>
                            <th class="px-8 py-5 text-xs font-bold text-slate-400 uppercase tracking-widest text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-slate-50"></tbody>
                    <tfoot class="bg-slate-50/80">
                        <tr>
                            <td colspan="3" class="px-8 py-6 text-right text-xs font-black text-slate-400 uppercase tracking-widest">Valor Total Acumulado</td>
                            <td id="grandTotal" class="px-8 py-6 text-right text-2xl font-black text-blue-600">R$ 0,00</td>
                            <td></td>
                        </tr>
                    </tfoot>
                </table>
            </div>
        </div>

        <!-- Rodapé -->
        <footer class="mt-12 mb-8 flex flex-col md:flex-row justify-between items-center gap-4 text-[11px] font-bold text-slate-400 uppercase tracking-widest">
            <div class="flex items-center gap-2">
                <span class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span>
                <p>© 2026 • Sistema de Gestão Hospitalar • Centro Cirúrgico</p>
            </div>
            <div class="flex items-center gap-6">
                <button id="btnClearAll" class="text-rose-400 hover:text-rose-600 transition-colors flex items-center gap-1">
                    Limpar Todos os Dados
                </button>
            </div>
        </footer>
    </div>

    <!-- Modal Usuários -->
    <div id="settingsModal">
        <div class="bg-white rounded-[2rem] shadow-2xl w-full max-w-2xl overflow-hidden flex flex-col border border-slate-100">
            <div class="p-8 border-b border-slate-50 flex justify-between items-center bg-slate-50/50">
                <h3 class="text-2xl font-black text-slate-800 tracking-tight">Utilizadores</h3>
                <button id="closeSettings" class="text-2xl font-bold">&times;</button>
            </div>
            <div class="p-8">
                <form id="newUserForm" class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
                    <input type="text" id="newUsername" required placeholder="Utilizador" class="px-4 py-2 border rounded-xl outline-none">
                    <input type="text" id="newPassword" required placeholder="Senha" class="px-4 py-2 border rounded-xl outline-none">
                    <button type="submit" class="bg-blue-600 text-white rounded-xl font-bold">Adicionar</button>
                </form>
                <div id="userListContainer" class="space-y-2"></div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, setDoc, query } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        let db, auth, appId;
        let isCloud = false;

        async function start() {
            // Tenta inicializar Firebase.
            try {
                if (typeof window.__firebase_config !== 'undefined' && window.__firebase_config) {
                    const config = JSON.parse(window.__firebase_config);
                    appId = typeof window.__app_id !== 'undefined' ? window.__app_id : 'default-app';
                    const app = initializeApp(config);
                    db = getFirestore(app);
                    auth = getAuth(app);

                    if (window.__initial_auth_token) {
                        await signInWithCustomToken(auth, window.__initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                    isCloud = true;
                }
            } catch (e) {
                console.warn("Usando modo offline (LocalStorage para dados)");
            }

            document.getElementById('initLoader').style.display = 'none';
            checkSession();
        }

        function checkSession() {
            // Alterado de localStorage para sessionStorage para exigir login ao fechar a aba
            const sessionActive = sessionStorage.getItem('clothes_sys_auth_session') === 'true';
            
            if (sessionActive) {
                showMain();
            } else {
                showLogin();
            }
        }

        function showLogin() {
            document.getElementById('loginScreen').style.display = 'flex';
            document.getElementById('mainContent').style.display = 'none';
        }

        function showMain() {
            document.getElementById('loginScreen').style.display = 'none';
            document.getElementById('mainContent').style.display = 'block';
            loadData();
        }

        function loadData() {
            if (isCloud) {
                onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), (snap) => {
                    const data = [];
                    snap.forEach(d => data.push({ id: d.id, ...d.data() }));
                    renderTable(data.sort((a,b) => b.createdAt - a.createdAt));
                });
                
                onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), (snap) => {
                    const users = [];
                    snap.forEach(d => users.push({ id: d.id, ...d.data() }));
                    renderUsers(users);
                });
            } else {
                const localData = JSON.parse(localStorage.getItem('clothes_data') || '[]');
                const localUsers = JSON.parse(localStorage.getItem('clothes_users') || '[]');
                renderTable(localData);
                renderUsers(localUsers);
            }
        }

        function renderTable(data) {
            const body = document.getElementById('tableBody');
            body.innerHTML = '';
            let total = 0;
            const price = parseFloat(document.getElementById('unitPriceInput').value);

            data.forEach(item => {
                const sub = item.qty * (item.priceAtTime || price);
                total += sub;
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="px-8 py-5 font-bold text-slate-700">${item.name}</td>
                    <td class="px-8 py-5 text-slate-500">${item.date}</td>
                    <td class="px-8 py-5 text-center">${item.qty}</td>
                    <td class="px-8 py-5 text-right font-black">R$ ${sub.toFixed(2)}</td>
                    <td class="px-8 py-5 text-center">
                        <button onclick="deleteRow('${item.id}')" class="text-rose-500 hover:underline">Excluir</button>
                    </td>
                `;
                body.appendChild(tr);
            });
            document.getElementById('grandTotal').innerText = `R$ ${total.toLocaleString('pt-BR', {minimumFractionDigits: 2})}`;
        }

        function renderUsers(users) {
            const container = document.getElementById('userListContainer');
            container.innerHTML = `<p class="text-xs font-bold text-slate-400 mb-2 uppercase">Acesso Mestre: CLX (02072007)</p>`;
            users.forEach(u => {
                const div = document.createElement('div');
                div.className = "flex justify-between p-3 bg-slate-50 rounded-xl text-sm border border-slate-100";
                div.innerHTML = `<span><b>${u.username}</b></span> <button onclick="deleteUser('${u.id}')" class="text-rose-500 font-bold">Remover</button>`;
                container.appendChild(div);
            });
        }

        // Globais
        window.deleteRow = async (id) => {
            if (!confirm("Confirmar exclusão deste registro?")) return;
            if (isCloud) {
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', id));
            } else {
                let data = JSON.parse(localStorage.getItem('clothes_data') || '[]');
                data = data.filter(i => i.id !== id);
                localStorage.setItem('clothes_data', JSON.stringify(data));
                loadData();
            }
        };

        window.deleteUser = async (id) => {
            if (!confirm("Remover este acesso secundário?")) return;
            if (isCloud) {
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'auth', id));
            } else {
                let users = JSON.parse(localStorage.getItem('clothes_users') || '[]');
                users = users.filter(u => u.id !== id);
                localStorage.setItem('clothes_users', JSON.stringify(users));
                loadData();
            }
        };

        // Eventos
        document.getElementById('loginForm').onsubmit = async (e) => {
            e.preventDefault();
            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            let ok = (u === "CLX" && p === "02072007");

            if (!ok) {
                let users = [];
                if (isCloud) {
                    const snap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'auth'));
                    users = snap.docs.map(d => d.data());
                } else {
                    users = JSON.parse(localStorage.getItem('clothes_users') || '[]');
                }
                if (users.find(user => user.username === u && user.password === p)) ok = true;
            }

            if (ok) {
                // Define a sessão ativa apenas para esta aba/janela aberta
                sessionStorage.setItem('clothes_sys_auth_session', 'true');
                showMain();
            } else {
                document.getElementById('loginError').classList.remove('hidden');
                setTimeout(() => document.getElementById('loginError').classList.add('hidden'), 3000);
            }
        };

        document.getElementById('entryForm').onsubmit = async (e) => {
            e.preventDefault();
            const newItem = {
                name: document.getElementById('patientName').value,
                date: document.getElementById('entryDate').value,
                qty: parseInt(document.getElementById('clothingQty').value),
                priceAtTime: parseFloat(document.getElementById('unitPriceInput').value),
                createdAt: Date.now()
            };

            if (isCloud) {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), newItem);
            } else {
                newItem.id = Date.now().toString();
                const data = JSON.parse(localStorage.getItem('clothes_data') || '[]');
                data.unshift(newItem);
                localStorage.setItem('clothes_data', JSON.stringify(data));
                loadData();
            }
            e.target.reset();
            document.getElementById('entryDate').valueAsDate = new Date();
        };

        document.getElementById('newUserForm').onsubmit = async (e) => {
            e.preventDefault();
            const u = {
                username: document.getElementById('newUsername').value,
                password: document.getElementById('newPassword').value
            };
            if (isCloud) {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), u);
            } else {
                u.id = Date.now().toString();
                const users = JSON.parse(localStorage.getItem('clothes_users') || '[]');
                users.push(u);
                localStorage.setItem('clothes_users', JSON.stringify(users));
                loadData();
            }
            e.target.reset();
        };

        document.getElementById('exportExcel').onclick = () => {
            const wb = XLSX.utils.table_to_book(document.getElementById("mainDataTable"));
            XLSX.writeFile(wb, "Relatorio_Roupas_CC.xlsx");
        };

        document.getElementById('btnClearAll').onclick = () => {
            if (confirm("ATENÇÃO: Deseja apagar todos os registros permanentemente?")) {
                if (!isCloud) {
                    localStorage.removeItem('clothes_data');
                    loadData();
                } else {
                    alert("A limpeza em massa na nuvem deve ser feita pelo administrador do banco.");
                }
            }
        };

        document.getElementById('btnLogoutAction').onclick = () => {
            sessionStorage.removeItem('clothes_sys_auth_session');
            location.reload();
        };

        document.getElementById('openSettings').onclick = () => document.getElementById('settingsModal').style.display = 'flex';
        document.getElementById('closeSettings').onclick = () => document.getElementById('settingsModal').style.display = 'none';
        
        // Setup inicial
        document.getElementById('entryDate').valueAsDate = new Date();
        start();
    </script>
</body>
</html>
