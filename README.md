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

        .glass-card {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(226, 232, 240, 0.8);
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

        input, select {
            transition: all 0.2s ease;
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

        .btn-primary:active {
            transform: translateY(0);
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

    <!-- Loader -->
    <div id="initLoader" class="loading-overlay">
        <div class="relative">
            <div class="w-16 h-16 border-4 border-blue-100 border-t-blue-600 rounded-full animate-spin"></div>
        </div>
        <p class="mt-4 text-slate-600 font-medium animate-pulse">Aceder ao sistema...</p>
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
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M10.293 3.293a1 1 0 011.414 0l6 6a1 1 0 010 1.414l-6 6a1 1 0 01-1.414-1.414L14.586 11H3a1 1 0 110-2h11.586l-4.293-4.293a1 1 0 010-1.414z" clip-rule="evenodd" />
                    </svg>
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
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                    </svg>
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

        <!-- Tabela de Dados -->
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
                    <tbody id="tableBody" class="divide-y divide-slate-50">
                        <!-- Conteúdo via JS -->
                    </tbody>
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

        <!-- RODAPÉ RESTAURADO -->
        <footer class="mt-12 mb-8 flex flex-col md:flex-row justify-between items-center gap-4 text-[11px] font-bold text-slate-400 uppercase tracking-widest">
            <div class="flex items-center gap-2">
                <span class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></span>
                <p>© 2026 • Sistema de Gestão Hospitalar • Centro Cirúrgico</p>
            </div>
            <div class="flex items-center gap-6">
                <button id="btnManualExport" class="hover:text-blue-500 transition-colors">Exportar Backup</button>
                <button id="btnClearAll" class="text-rose-400 hover:text-rose-600 transition-colors flex items-center gap-1">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                    </svg>
                    Limpar Todos os Dados
                </button>
            </div>
        </footer>
    </div>

    <!-- Modal Configurações -->
    <div id="settingsModal">
        <div class="bg-white rounded-[2rem] shadow-2xl w-full max-w-2xl overflow-hidden flex flex-col border border-slate-100">
            <div class="p-8 border-b border-slate-50 flex justify-between items-center bg-slate-50/50">
                <div>
                    <h3 class="text-2xl font-black text-slate-800 tracking-tight">Utilizadores</h3>
                    <p class="text-sm text-slate-500 font-medium">Gerencie quem tem acesso ao sistema</p>
                </div>
                <button id="closeSettings" class="w-12 h-12 flex items-center justify-center rounded-2xl bg-white border border-slate-200 text-slate-400 hover:text-slate-600 transition-all shadow-sm text-2xl">&times;</button>
            </div>
            
            <div class="p-8 overflow-y-auto max-h-[70vh]">
                <form id="newUserForm" class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8 p-6 bg-blue-50/50 rounded-3xl border border-blue-100">
                    <input type="text" id="newUsername" required placeholder="Utilizador" class="px-4 py-3 rounded-xl border border-slate-200 outline-none text-sm">
                    <input type="text" id="newPassword" required placeholder="Senha" class="px-4 py-3 rounded-xl border border-slate-200 outline-none text-sm">
                    <button type="submit" class="bg-blue-600 text-white font-bold py-3 rounded-xl text-sm hover:bg-blue-700 transition-all">Novo Acesso</button>
                </form>

                <div class="space-y-3" id="userListContainer">
                    <!-- Lista de usuários -->
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, setDoc, getDoc, query } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        let db, auth, appId;
        let currentUser = null;

        // Inicialização com tolerância a erros de injeção
        async function start() {
            const configWait = setInterval(() => {
                if (typeof window.__firebase_config !== 'undefined') {
                    clearInterval(configWait);
                    initFirebase();
                }
            }, 200);

            // Timeout de segurança para não travar a UI
            setTimeout(() => {
                clearInterval(configWait);
                document.getElementById('initLoader').style.opacity = '0';
                setTimeout(() => document.getElementById('initLoader').style.display = 'none', 400);
                checkLocalSession();
            }, 1500);
        }

        async function initFirebase() {
            try {
                const config = JSON.parse(window.__firebase_config);
                appId = typeof window.__app_id !== 'undefined' ? window.__app_id : 'default-app-id';
                const app = initializeApp(config);
                db = getFirestore(app);
                auth = getAuth(app);

                if (window.__initial_auth_token) {
                    await signInWithCustomToken(auth, window.__initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    currentUser = user;
                    if (user && localStorage.getItem('clothes_sys_auth') === 'true') {
                        setupData();
                    }
                });
            } catch (e) {
                console.error("Firebase fail", e);
            }
        }

        function checkLocalSession() {
            const logged = localStorage.getItem('clothes_sys_auth') === 'true';
            if (logged) showMain(); else showLogin();
        }

        function showLogin() {
            document.getElementById('loginScreen').style.display = 'flex';
            document.getElementById('mainContent').style.display = 'none';
        }

        function showMain() {
            document.getElementById('loginScreen').style.display = 'none';
            document.getElementById('mainContent').style.display = 'block';
            if (db) setupData();
        }

        function setupData() {
            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), (snap) => {
                const list = [];
                snap.forEach(d => list.push({ id: d.id, ...d.data() }));
                list.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));
                renderTable(list);
            }, (err) => console.error(err));

            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), (snap) => {
                const container = document.getElementById('userListContainer');
                container.innerHTML = `
                    <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border border-slate-100">
                        <div>
                            <p class="font-bold text-slate-800">CLX <span class="text-[10px] bg-blue-100 text-blue-600 px-2 py-0.5 rounded-full ml-2 uppercase">Admin</span></p>
                            <p class="text-xs text-slate-400">Acesso mestre</p>
                        </div>
                        <span class="text-slate-300">••••••••</span>
                    </div>
                `;
                snap.forEach(d => {
                    const div = document.createElement('div');
                    div.className = "flex items-center justify-between p-4 bg-white border border-slate-100 rounded-2xl shadow-sm";
                    div.innerHTML = `
                        <div>
                            <p class="font-bold text-slate-800">${d.data().username}</p>
                            <p class="text-xs text-slate-400">Acesso restrito</p>
                        </div>
                        <div class="flex items-center gap-4">
                            <span class="text-slate-400 font-mono text-xs">${d.data().password}</span>
                            <button onclick="deleteUser('${d.id}')" class="text-rose-500 hover:bg-rose-50 p-2 rounded-xl transition-all">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                            </button>
                        </div>
                    `;
                    container.appendChild(div);
                });
            }, (err) => console.error(err));
        }

        function renderTable(data) {
            const body = document.getElementById('tableBody');
            body.innerHTML = '';
            let total = 0;
            const currentPrice = parseFloat(document.getElementById('unitPriceInput').value) || 0;

            data.forEach(item => {
                const sub = item.qty * (item.priceAtTime || currentPrice);
                total += sub;
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50/50 transition-colors group";
                tr.innerHTML = `
                    <td class="px-8 py-5 font-bold text-slate-700">${item.name}</td>
                    <td class="px-8 py-5 text-slate-500 font-medium">${item.date}</td>
                    <td class="px-8 py-5 text-center"><span class="bg-slate-100 text-slate-700 px-3 py-1 rounded-lg font-black text-xs">${item.qty}</span></td>
                    <td class="px-8 py-5 text-right font-black text-slate-800">R$ ${sub.toLocaleString('pt-BR', {minimumFractionDigits: 2})}</td>
                    <td class="px-8 py-5 text-center">
                        <button onclick="deleteRecord('${item.id}')" class="text-slate-300 hover:text-rose-500 transition-all opacity-0 group-hover:opacity-100">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                        </button>
                    </td>
                `;
                body.appendChild(tr);
            });
            document.getElementById('grandTotal').innerText = `R$ ${total.toLocaleString('pt-BR', {minimumFractionDigits: 2})}`;
        }

        // Ações Globais
        window.deleteRecord = async (id) => { if(confirm("Deseja eliminar este registo?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', id)); };
        window.deleteUser = async (id) => { if(confirm("Remover este acesso?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'auth', id)); };

        document.getElementById('loginForm').onsubmit = async (e) => {
            e.preventDefault();
            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            let ok = (u === "CLX" && p === "02072007");

            if (!ok && db) {
                const snap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'auth'));
                snap.forEach(d => { if(d.data().username === u && d.data().password === p) ok = true; });
            }

            if (ok) { localStorage.setItem('clothes_sys_auth', 'true'); showMain(); }
            else { 
                document.getElementById('loginError').classList.remove('hidden');
                setTimeout(() => document.getElementById('loginError').classList.add('hidden'), 3000);
            }
        };

        document.getElementById('entryForm').onsubmit = async (e) => {
            e.preventDefault();
            if(!db) return;
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

        document.getElementById('exportExcel').onclick = () => {
            const wb = XLSX.utils.table_to_book(document.getElementById("mainDataTable"));
            XLSX.writeFile(wb, "Relatorio_Centro_Cirurgico.xlsx");
        };

        document.getElementById('btnClearAll').onclick = async () => {
            if(confirm("ATENÇÃO: Deseja apagar ABSOLUTAMENTE TODOS os registros? Esta ação não pode ser desfeita.")) {
                const snap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'registros'));
                snap.forEach(async (d) => {
                    await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', d.id));
                });
            }
        };

        document.getElementById('btnLogoutAction').onclick = () => { localStorage.removeItem('clothes_sys_auth'); location.reload(); };
        document.getElementById('openSettings').onclick = () => document.getElementById('settingsModal').style.display = 'flex';
        document.getElementById('closeSettings').onclick = () => document.getElementById('settingsModal').style.display = 'none';
        document.getElementById('entryDate').valueAsDate = new Date();
        
        start();
    </script>
</body>
</html>
