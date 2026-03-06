<!DOCTYPE html>
<html lang="pt-pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bússola Ética - CerimmEventos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .compass-transition { transition: transform 3s cubic-bezier(0.15, 0, 0.15, 1); }
        .word-cell.found { background-color: #22c55e !important; color: white !important; transform: scale(1.1); box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
        .word-cell.selected { background-color: #60a5fa !important; color: white !important; ring: 4px #bfdbfe; }
        @media print { .no-print { display: none; } }
        #error-message { display: none; }
    </style>
</head>
<body class="bg-gradient-to-br from-slate-100 to-blue-50 min-h-screen p-4 md:p-8 flex flex-col items-center">

    <header class="text-center mb-8 no-print">
        <div class="inline-block px-4 py-1 bg-blue-600 text-white rounded-full text-xs font-bold mb-3 tracking-widest uppercase">
            CerimmEventos Apresenta
        </div>
        <h1 class="text-4xl md:text-5xl font-black text-slate-800 tracking-tight">Bússola Ética</h1>
        <div class="h-1 w-24 bg-blue-600 mx-auto mt-2 rounded-full"></div>
    </header>

    <div id="error-message" class="w-full max-w-4xl mb-6 p-4 bg-red-100 border-l-4 border-red-500 text-red-700 rounded-lg shadow-sm no-print">
        <div class="flex items-center gap-2 font-bold mb-1">
            <i data-lucide="alert-circle"></i> Atenção: Configuração Necessária
        </div>
        <p class="text-sm">A ligação ao banco de dados falhou. <b>Ação necessária:</b> Vá ao Firebase Console > Authentication > Sign-in method e ative o "Anónimo". Sem isto, os resultados não serão gravados.</p>
        <button onclick="renderStart()" class="mt-2 text-xs underline font-bold">Continuar sem gravar dados</button>
    </div>

    <main id="app-container" class="w-full max-w-6xl">
        <div class="flex flex-col items-center justify-center p-20 animate-pulse text-slate-400">
            <i data-lucide="compass" class="w-16 h-16 mb-4"></i>
            <p class="font-bold">A ligar ao servidor...</p>
        </div>
    </main>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyDfpgof4Snkd2KIfyZl5ZhGoJg1nM0VskU",
            authDomain: "bussolaetica.firebaseapp.com",
            projectId: "bussolaetica",
            storageBucket: "bussolaetica.firebasestorage.app",
            messagingSenderId: "104879930878",
            appId: "1:104879930878:web:95900a5206d720abcc14af",
            measurementId: "G-69TPBFJBJT"
        };

        const appId = "bussolaetica";
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let user = null;
        let authHandled = false;

        const initAuth = async () => {
            try {
                await signInAnonymously(auth);
            } catch (error) {
                console.error("Firebase Auth Error:", error);
                document.getElementById('error-message').style.display = 'block';
                if (!authHandled) renderStart(); 
            }
        };

        onAuthStateChanged(auth, (u) => {
            user = u;
            if (u && !authHandled) {
                authHandled = true;
                renderStart();
            }
        });

        // Timeout para não ficar preso no loading se o Firebase falhar
        setTimeout(() => {
            if (!authHandled) {
                document.getElementById('error-message').style.display = 'block';
                renderStart();
            }
        }, 5000);

        initAuth();

        window.saveResultToFirebase = async (data) => {
            if (!user) return;
            try {
                const resultsCol = collection(db, 'artifacts', appId, 'public', 'data', 'results');
                await addDoc(resultsCol, { ...data, userId: user.uid, timestamp: serverTimestamp() });
            } catch (e) { console.error(e); }
        };

        const themes = {
            Norte: { title: "Integridade e Honestidade", icon: 'shield-check', color: "bg-blue-600", lightColor: "bg-blue-50 border-blue-600 text-blue-900", questions: [ { q: "Cometeu um erro que ninguém percebeu, mas que pode afetar o stock futuro. Qual a atitude correta?", options: [ { text: "Assumir o erro e relatar para correção imediata.", correct: true }, { text: "Ficar em silêncio, já que ninguém viu e pode resolver-se sozinho.", correct: false } ], explanation: "A honestidade é a base da confiança institucional." }, { q: "Um cliente antigo pede um 'favor' que desvia levemente da regra para agilizar um processo. Deve:", options: [ { text: "Negar educadamente e seguir o padrão ético da empresa.", correct: true }, { text: "Atender o pedido para manter o bom relacionamento com o cliente.", correct: false } ], explanation: "A integridade não abre exceções por amizade." } ] },
            Sul: { title: "Assédio no Trabalho", icon: 'user-x', color: "bg-red-600", lightColor: "bg-red-50 border-red-600 text-red-900", questions: [ { q: "Incentivar 'alcunhas' pejorativas ou piadas sobre a aparência de um colega é considerado:", options: [ { text: "Assédio Moral, pois gera um ambiente hostil.", correct: true }, { text: "Apenas integração da equipa.", correct: false } ], explanation: "O respeito pela dignidade humana é inegociável." }, { q: "Se presencia um ato de desrespeito de um gestor contra um colega, deve:", options: [ { text: "Denunciar o ocorrido nos canais de ética.", correct: true }, { text: "Não se envolver para evitar problemas.", correct: false } ], explanation: "A omissão também fere a ética." } ] },
            Leste: { title: "Recursos e Informações", icon: 'monitor', color: "bg-emerald-600", lightColor: "bg-emerald-50 border-emerald-600 text-emerald-900", questions: [ { q: "Levar materiais de escritório para uso escolar dos filhos em casa é permitido?", options: [ { text: "Não, os recursos são exclusivos para fins profissionais.", correct: true }, { text: "Sim, se for em pequena quantidade.", correct: false } ], explanation: "O uso indevido de bens é uma falha de conduta." }, { q: "Partilhar a sua palavra-passe de acesso com um colega é:", options: [ { text: "Uma violação grave de segurança.", correct: true }, { text: "Uma demonstração de trabalho em equipa.", correct: false } ], explanation: "As palavras-passe são pessoais e intransferíveis." } ] },
            Oeste: { title: "Normas e Responsabilidades", icon: 'clipboard-check', color: "bg-orange-600", lightColor: "bg-orange-50 border-orange-600 text-orange-900", questions: [ { q: "Ao notar uma falha de segurança num processo, a sua primeira ação deve ser:", options: [ { text: "Seguir os protocolos e reportar a falha.", correct: true }, { text: "Ignorar para não atrasar a entrega.", correct: false } ], explanation: "O cumprimento das normas garante a segurança de todos." }, { q: "O Código de Ética deve ser lido e seguido por:", options: [ { text: "Todos os colaboradores.", correct: true }, { text: "Apenas pelos novos funcionários.", correct: false } ], explanation: "A ética é universal na instituição." } ] }
        };

        let score = { correct: 0, total: 0 };
        let answeredCount = { Norte: 0, Sul: 0, Leste: 0, Oeste: 0 };
        let rotation = 0;
        let isSpinning = false;
        const totalTarget = 8;

        window.renderStart = function() {
            const container = document.getElementById('app-container');
            container.innerHTML = `
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-12 items-center animate-in fade-in duration-700">
                    <div class="relative flex flex-col items-center">
                        <div class="absolute -top-10 left-1/2 -translate-x-1/2 font-black text-blue-600 flex flex-col items-center"><i data-lucide="shield-check"></i><span class="text-xs uppercase">Integridade</span></div>
                        <div class="absolute -bottom-10 left-1/2 -translate-x-1/2 font-black text-red-600 flex flex-col items-center"><i data-lucide="user-x"></i><span class="text-xs uppercase">Assédio</span></div>
                        <div class="absolute top-1/2 -right-24 -translate-y-1/2 font-black text-emerald-600 hidden md:flex flex-col items-center"><i data-lucide="monitor"></i><span class="text-xs uppercase">Recursos</span></div>
                        <div class="absolute top-1/2 -left-24 -translate-y-1/2 font-black text-orange-600 hidden md:flex flex-col items-center"><i data-lucide="clipboard-check"></i><span class="text-xs uppercase">Normas</span></div>
                        <div id="compass" class="w-64 h-64 md:w-80 md:h-80 rounded-full border-[12px] border-slate-800 shadow-2xl relative bg-white overflow-hidden compass-transition">
                            <div class="absolute top-0 left-0 w-1/2 h-1/2 bg-blue-500 flex items-center justify-center border-r border-b border-white/20"><span class="text-white font-black text-2xl -rotate-45">N</span></div>
                            <div class="absolute top-0 right-0 w-1/2 h-1/2 bg-emerald-500 flex items-center justify-center border-b border-white/20"><span class="text-white font-black text-2xl rotate-45">L</span></div>
                            <div class="absolute bottom-0 left-0 w-1/2 h-1/2 bg-orange-500 flex items-center justify-center border-r border-white/20"><span class="text-white font-black text-2xl rotate-[225deg]">O</span></div>
                            <div class="absolute bottom-0 right-0 w-1/2 h-1/2 bg-red-500 flex items-center justify-center"><span class="text-white font-black text-2xl rotate-[135deg]">S</span></div>
                            <div class="absolute inset-0 flex items-center justify-center z-20"><div class="w-6 h-6 bg-slate-900 rounded-full border-4 border-white shadow-lg"><div class="absolute bottom-full left-1/2 -translate-x-1/2 w-3 h-28 md:h-36 bg-gradient-to-t from-slate-900 to-red-400 rounded-t-full"></div></div></div>
                        </div>
                        <button onclick="handleSpin()" id="spin-btn" class="mt-16 px-10 py-5 bg-slate-900 text-white rounded-2xl font-black text-xl flex items-center gap-3 shadow-xl active:scale-95 transition-all"><i data-lucide="rotate-cw" id="spin-icon"></i> <span id="spin-text">Girar a Bússola</span></button>
                        <div class="mt-8 flex gap-6">
                            <div class="text-center"><p class="text-xs text-slate-400 font-bold uppercase">Questões</p><p class="text-2xl font-black text-slate-700">${score.total} / ${totalTarget}</p></div>
                            <div class="w-px h-10 bg-slate-200"></div>
                            <div class="text-center"><p class="text-xs text-slate-400 font-bold uppercase">Acertos</p><p class="text-2xl font-black text-green-600">${score.correct}</p></div>
                        </div>
                    </div>
                    <div id="interaction-panel" class="w-full">
                        <div class="bg-white border-4 border-dashed border-slate-200 rounded-[2.5rem] p-12 text-center flex flex-col items-center">
                            <div class="w-20 h-20 bg-blue-50 text-blue-600 rounded-full flex items-center justify-center mb-6"><i data-lucide="compass" class="w-10 h-10"></i></div>
                            <h2 class="text-2xl font-bold text-slate-800 mb-2">Pronto a começar?</h2>
                            <p class="text-slate-500">Gire a bússola para navegar pelos valores da nossa instituição.</p>
                        </div>
                    </div>
                </div>
            `;
            lucide.createIcons();
        }

        window.handleSpin = function() {
            if (isSpinning || score.total >= totalTarget) return;
            isSpinning = true;
            document.getElementById('spin-icon').classList.add('animate-spin');
            document.getElementById('spin-text').innerText = "Navegando...";
            rotation += (4 + Math.floor(Math.random() * 4)) * 360 + Math.floor(Math.random() * 360);
            document.getElementById('compass').style.transform = `rotate(${rotation}deg)`;
            setTimeout(() => {
                isSpinning = false;
                const finalAngle = rotation % 360;
                let point = (finalAngle >= 315 || finalAngle < 45) ? "Norte" : (finalAngle >= 45 && finalAngle < 135) ? "Leste" : (finalAngle >= 135 && finalAngle < 225) ? "Sul" : "Oeste";
                renderQuestion(point);
            }, 3000);
        }

        function renderQuestion(point) {
            const theme = themes[point];
            const qIndex = answeredCount[point] < 2 ? answeredCount[point] : Math.floor(Math.random() * 2);
            const question = theme.questions[qIndex];
            document.getElementById('interaction-panel').innerHTML = `
                <div class="rounded-[2.5rem] p-8 shadow-2xl flex flex-col border-b-8 ${theme.lightColor} animate-in fade-in duration-500">
                    <div class="flex items-center gap-3 mb-6"><div class="p-3 bg-white rounded-xl shadow-sm"><i data-lucide="${theme.icon}"></i></div><h3 class="text-xl font-black uppercase tracking-tight">${theme.title}</h3></div>
                    <p class="text-xl font-bold mb-8 leading-tight">${question.q}</p>
                    <div class="space-y-4">
                        ${question.options.map((opt, i) => `<button onclick="handleAnswer('${point}', ${qIndex}, ${i})" class="w-full text-left p-5 rounded-2xl border-2 font-bold bg-white border-slate-200 hover:border-slate-800 transition-all"><span>${opt.text}</span></button>`).join('')}
                    </div>
                </div>
            `;
            lucide.createIcons();
        }

        window.handleAnswer = function(point, qIdx, optIdx) {
            const question = themes[point].questions[qIdx];
            const isCorrect = question.options[optIdx].correct;
            score.total++; if (isCorrect) score.correct++; answeredCount[point]++;
            document.getElementById('interaction-panel').innerHTML = `
                <div class="rounded-[2.5rem] p-8 shadow-2xl flex flex-col border-b-8 ${isCorrect ? 'bg-green-50 border-green-600' : 'bg-red-50 border-red-600'}">
                    <div class="flex items-center gap-3 mb-4"><i data-lucide="${isCorrect ? 'check-circle-2' : 'x-circle'}" class="${isCorrect ? 'text-green-600' : 'text-red-600'}"></i><h3 class="text-xl font-black uppercase">${isCorrect ? 'Resposta Correta!' : 'Atenção ao Código'}</h3></div>
                    <div class="p-5 bg-slate-900 text-white rounded-2xl flex flex-col gap-3">
                        <div class="flex items-center gap-2 text-blue-400 font-bold text-sm uppercase"><i data-lucide="info" class="w-4 h-4"></i> Reflexão Ética</div>
                        <p class="text-sm opacity-90">${question.explanation}</p>
                        <button onclick="${score.total >= totalTarget ? 'renderResult()' : 'renderStart()'}" class="mt-2 w-full py-3 bg-blue-600 hover:bg-blue-500 rounded-xl font-bold transition-colors">${score.total >= totalTarget ? 'Ver Relatório Final' : 'Próximo Dilema'}</button>
                    </div>
                </div>
            `;
            lucide.createIcons();
        }

        window.renderResult = function() {
            window.saveResultToFirebase({ correct: score.correct, total: score.total, status: (score.correct/score.total) >= 0.7 ? "Excelência" : "Atenção" });
            const performance = score.correct / score.total;
            document.getElementById('app-container').innerHTML = `
                <div class="flex flex-col items-center w-full max-w-4xl mx-auto animate-in zoom-in duration-500">
                    <div class="bg-white border-[12px] border-double border-slate-200 p-8 md:p-16 rounded-[3rem] shadow-2xl w-full text-center">
                        <div class="w-20 h-20 bg-blue-600 text-white rounded-full flex items-center justify-center mx-auto mb-6 shadow-xl"><i data-lucide="award" class="w-10 h-10"></i></div>
                        <h2 class="text-4xl font-black text-slate-800 mb-2 uppercase tracking-tighter">Certificado Ético</h2>
                        <div class="my-10 py-8 border-y border-slate-100 bg-slate-50/50">
                            <div class="flex justify-center gap-12 mt-8">
                                <div class="text-center"><span class="block text-5xl font-black text-green-600">${score.correct}</span><span class="text-xs font-bold uppercase text-slate-400">Conformidade</span></div>
                                <div class="text-center"><span class="block text-5xl font-black text-red-400">${score.total - score.correct}</span><span class="text-xs font-bold uppercase text-slate-400">Melhorias</span></div>
                            </div>
                        </div>
                        <div class="inline-block px-10 py-5 rounded-2xl font-black text-xl mb-12 text-white ${performance >= 0.7 ? 'bg-green-500' : 'bg-orange-500'}">${performance >= 0.7 ? "✓ EXCELÊNCIA ÉTICA" : "⚠ NECESSÁRIO REFORÇO"}</div>
                        <div class="flex flex-col md:flex-row gap-4 justify-center no-print">
                            <button onclick="renderWordSearch()" class="bg-blue-600 text-white px-8 py-4 rounded-2xl font-black flex items-center justify-center gap-2 hover:bg-blue-700 shadow-lg"><i data-lucide="mouse-pointer-2"></i> Caça-Palavras Manual</button>
                            <button onclick="window.print()" class="border-2 border-slate-200 px-8 py-4 rounded-2xl font-black flex items-center justify-center gap-2 hover:bg-slate-50"><i data-lucide="download"></i> Salvar PDF</button>
                        </div>
                    </div>
                </div>
            `;
            lucide.createIcons();
        }

        const targetWords = [ { word: "ASSEDIO", cells: [[0,0], [0,1], [0,2], [0,3], [0,4], [0,5], [0,6]] }, { word: "ETICA", cells: [[2,3], [2,4], [2,5], [2,6], [2,7]] }, { word: "MORAL", cells: [[3,0], [3,1], [3,2], [3,3], [3,4]] }, { word: "NORMAS", cells: [[5,1], [5,2], [5,3], [5,4], [5,5], [5,6]] } ];
        let foundWords = [], selectedCells = [];

        window.renderWordSearch = function() {
            const alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
            const baseGrid = [ ['A','S','S','E','D','I','O','X','X'], ['X','X','X','X','X','X','X','X','X'], ['X','X','X','E','T','I','C','A','X'], ['M','O','R','A','L','X','X','X','X'], ['X','X','X','X','X','X','X','X','X'], ['X','N','O','R','M','A','S','X','X'], ['X','X','X','X','X','X','X','X','X'] ];
            const gridHTML = baseGrid.map((row, r) => row.map((char, c) => `<button onclick="toggleCell(${r}, ${c})" id="cell-${r}-${c}" class="word-cell w-8 h-8 md:w-12 md:h-12 flex items-center justify-center rounded-full font-black text-lg bg-slate-700 text-slate-300 hover:bg-slate-600 transition-all">${char === 'X' ? alphabet[Math.floor(Math.random() * alphabet.length)] : char}</button>`).join('')).join('');
            document.getElementById('app-container').innerHTML = `
                <div class="flex flex-col items-center bg-white p-6 md:p-10 rounded-[3rem] shadow-2xl w-full max-w-2xl mx-auto border-4 border-blue-100 animate-in slide-in-from-right-10 duration-500">
                    <div class="flex items-center gap-3 mb-4"><div class="p-2 bg-blue-600 text-white rounded-lg"><i data-lucide="search"></i></div><h3 class="text-3xl font-black text-slate-800">Caça-Palavras Ético</h3></div>
                    <p id="wordsearch-hint" class="text-slate-500 mb-6 text-center font-medium">Encontre as palavras ASSEDIO, MORAL, ETICA e NORMAS!</p>
                    <div class="bg-slate-800 p-4 rounded-3xl shadow-inner mb-8"><div class="grid grid-cols-9 gap-2">${gridHTML}</div></div>
                    <div class="flex flex-wrap justify-center gap-3" id="word-list">
                        ${targetWords.map(tw => `<div id="label-${tw.word}" class="px-4 py-2 rounded-xl font-bold border-2 bg-slate-50 border-slate-200 text-slate-300 flex items-center gap-2">${tw.word}</div>`).join('')}
                    </div>
                </div>
            `;
            lucide.createIcons();
        }

        window.toggleCell = function(r, c) {
            const cellElem = document.getElementById(`cell-${r}-${c}`);
            if (cellElem.classList.contains('found')) return;
            if (cellElem.classList.contains('selected')) {
                cellElem.classList.remove('selected');
                selectedCells = selectedCells.filter(cell => !(cell[0] === r && cell[1] === c));
            } else {
                cellElem.classList.add('selected');
                selectedCells.push([r, c]);
            }
            targetWords.forEach(tw => {
                if (!foundWords.includes(tw.word)) {
                    const allSelected = tw.cells.every(twCell => selectedCells.some(sCell => sCell[0] === twCell[0] && sCell[1] === twCell[1]));
                    if (allSelected && selectedCells.length === tw.cells.length) {
                        foundWords.push(tw.word);
                        tw.cells.forEach(cell => { const el = document.getElementById(`cell-${cell[0]}-${cell[1]}`); el.classList.remove('selected'); el.classList.add('found'); });
                        const label = document.getElementById(`label-${tw.word}`);
                        label.className = "px-4 py-2 rounded-xl font-bold border-2 bg-green-50 border-green-500 text-green-700 flex items-center gap-2";
                        label.innerHTML = `<i data-lucide="check-circle-2" class="w-4 h-4"></i> ${tw.word}`;
                        selectedCells = []; lucide.createIcons();
                        if (foundWords.length === targetWords.length) document.getElementById('wordsearch-hint').innerHTML = `<span class="text-green-600 font-bold">🎉 Incrível! Todos os conceitos foram fixados!</span>`;
                    }
                }
            });
        }
    </script>
</body>
</html>
