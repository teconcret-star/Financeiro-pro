<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financeiro Pro - Versão Final</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { background-color: #f3f4f6; font-family: sans-serif; }
        .card { background: white; padding: 1.2rem; border-radius: 1rem; box-shadow: 0 2px 5px rgba(0,0,0,0.05); transition: all 0.3s; }
        .calendar-day { min-height: 45px; display: flex; flex-direction: column; align-items: center; justify-content: center; font-size: 0.75rem; border-radius: 0.5rem; border: 1px solid #e5e7eb; cursor: pointer; transition: 0.2s; }
        .calendar-day:hover { background-color: #fef08a; transform: scale(1.05); }
        .has-receita { background-color: #dcfce7 !important; border-color: #22c55e !important; color: #166534; font-weight: bold; }
        .has-despesa { background-color: #fee2e2 !important; border-color: #ef4444 !important; color: #991b1b; font-weight: bold; }
        .has-ambos { background-image: linear-gradient(135deg, #dcfce7 50%, #fee2e2 50%) !important; font-weight: bold; }
        .today { border: 2px solid #2563eb !important; text-decoration: underline; }
    </style>
</head>
<body class="p-4 pb-24">

    <div class="flex justify-between items-center mb-6">
        <h1 class="text-xl font-bold text-gray-800">Minhas Contas 🏠</h1>
        <div class="flex gap-2">
            <label class="bg-gray-700 text-white text-[10px] font-bold px-3 py-2 rounded-lg shadow-md cursor-pointer hover:bg-gray-800 active:scale-95">
                📥 IMPORTAR <input type="file" id="importFile" class="hidden" accept=".csv" onchange="importarExcel(event)">
            </label>
            <button onclick="exportarExcel()" class="bg-blue-600 text-white text-[10px] font-bold px-3 py-2 rounded-lg shadow-md hover:bg-blue-700 active:scale-95">
                📊 EXPORTAR
            </button>
        </div>
    </div>

    <div class="card mb-6">
        <div class="flex justify-between items-center mb-4">
            <h3 id="mes-ano" class="font-bold text-gray-700 capitalize text-sm"></h3>
            <span class="text-[9px] text-gray-400">Clique em um dia para lançar</span>
        </div>
        <div class="grid grid-cols-7 gap-1 text-center text-[9px] font-bold text-gray-400 mb-2">
            <span>DOM</span><span>SEG</span><span>TER</span><span>QUA</span><span>QUI</span><span>SEX</span><span>SÁB</span>
        </div>
        <div id="calendar-grid" class="grid grid-cols-7 gap-1"></div>
    </div>

    <div class="grid grid-cols-2 gap-3 mb-4 text-center">
        <div class="card bg-green-50 border-b-4 border-green-500">
            <p class="text-[10px] text-green-600 font-bold uppercase">Entradas</p>
            <p id="total-receitas" class="text-lg font-bold text-green-700">R$ 0,00</p>
        </div>
        <div class="card bg-red-50 border-b-4 border-red-500">
            <p class="text-[10px] text-red-600 font-bold uppercase">Saídas</p>
            <p id="total-despesas" class="text-lg font-bold text-red-700">R$ 0,00</p>
        </div>
    </div>

    <div id="form-container" class="card mb-6 border-t-4 border-gray-800">
        <input type="hidden" id="edit-id">
        <input type="text" id="desc" placeholder="Descrição" class="w-full border p-3 rounded-lg mb-2 outline-none focus:ring-2 focus:ring-blue-500">
        <input type="number" id="valor" placeholder="Valor R$" class="w-full border p-3 rounded-lg mb-2 outline-none">
        
        <div class="grid grid-cols-2 gap-2 mb-4">
            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Vencimento</label>
                <input type="date" id="dataVenc" class="w-full border p-2 rounded-lg outline-none text-sm bg-gray-50 transition-all">
            </div>
            <div>
                <label class="text-[10px] font-bold text-gray-400 uppercase ml-1">Status</label>
                <select id="status" class="w-full border p-2 rounded-lg outline-none text-sm bg-gray-50">
                    <option value="pago">Pago</option>
                    <option value="pendente">Pendente</option>
                </select>
            </div>
        </div>
        
        <div class="flex gap-2 text-sm font-bold">
            <button onclick="salvar('receita')" id="btn-receita" class="flex-1 bg-green-600 text-white py-3 rounded-xl uppercase hover:bg-green-700"> + Receita</button>
            <button onclick="salvar('despesa')" id="btn-despesa" class="flex-1 bg-red-600 text-white py-3 rounded-xl uppercase hover:bg-red-700">- Despesa</button>
            <button onclick="cancelarEdicao()" id="btn-cancelar" class="hidden flex-1 bg-gray-400 text-white py-3 rounded-xl uppercase">Cancelar</button>
        </div>
    </div>

    <div id="lista" class="space-y-3"></div>

    <script>
        let transacoes = JSON.parse(localStorage.getItem('finance_app_v4')) || [];

        function salvar(tipo) {
            const idEdit = document.getElementById('edit-id').value;
            const desc = document.getElementById('desc').value;
            const valor = parseFloat(document.getElementById('valor').value);
            const dataVenc = document.getElementById('dataVenc').value;
            const status = document.getElementById('status').value;

            if (!desc || isNaN(valor) || !dataVenc) return alert("Preencha todos os campos!");

            if (idEdit) {
                const index = transacoes.findIndex(t => t.id == idEdit);
                transacoes[index] = { ...transacoes[index], desc, valor, dataVenc, status };
            } else {
                transacoes.push({ id: Date.now(), desc, valor, tipo, dataVenc, status });
            }
            cancelarEdicao();
            atualizarApp();
        }

        function editar(id) {
            const item = transacoes.find(t => t.id === id);
            document.getElementById('edit-id').value = item.id;
            document.getElementById('desc').value = item.desc;
            document.getElementById('valor').value = item.valor;
            document.getElementById('dataVenc').value = item.dataVenc;
            document.getElementById('status').value = item.status;
            
            document.getElementById('form-container').classList.add('ring-2', 'ring-blue-500');
            document.getElementById('btn-receita').innerText = "CONCLUIR";
            document.getElementById('btn-despesa').classList.add('hidden');
            document.getElementById('btn-cancelar').classList.remove('hidden');
            window.scrollTo({top: 0, behavior: 'smooth'});
        }

        function cancelarEdicao() {
            document.getElementById('edit-id').value = '';
            document.getElementById('desc').value = '';
            document.getElementById('valor').value = '';
            document.getElementById('dataVenc').value = '';
            document.getElementById('btn-receita').innerText = "+ Receita";
            document.getElementById('btn-despesa').classList.remove('hidden');
            document.getElementById('btn-cancelar').classList.add('hidden');
            document.getElementById('form-container').classList.remove('ring-2', 'ring-blue-500');
        }

        function remover(id) {
            if(confirm("Deseja realmente excluir este registro?")) {
                transacoes = transacoes.filter(t => t.id !== id);
                atualizarApp();
            }
        }

        function focarData(data) {
            const input = document.getElementById('dataVenc');
            input.value = data;
            window.scrollTo({top: document.getElementById('form-container').offsetTop - 20, behavior: 'smooth'});
            input.classList.add('ring-4', 'ring-blue-300');
            setTimeout(() => input.classList.remove('ring-4', 'ring-blue-300'), 1500);
        }

        function gerarCalendario() {
            const grid = document.getElementById('calendar-grid');
            const titulo = document.getElementById('mes-ano');
            grid.innerHTML = '';
            const hoje = new Date();
            const mes = hoje.getMonth();
            const ano = hoje.getFullYear();
            titulo.innerText = hoje.toLocaleString('pt-br', { month: 'long', year: 'numeric' });
            
            const primeiroDia = new Date(ano, mes, 1).getDay();
            const totalDias = new Date(ano, mes + 1, 0).getDate();

            for (let i = 0; i < primeiroDia; i++) grid.innerHTML += `<div></div>`;

            for (let dia = 1; dia <= totalDias; dia++) {
                const dataString = `${ano}-${String(mes + 1).padStart(2, '0')}-${String(dia).padStart(2, '0')}`;
                const movs = transacoes.filter(t => t.dataVenc === dataString);
                
                let classeCor = "";
                const temRec = movs.some(t => t.tipo === 'receita');
                const temDes = movs.some(t => t.tipo === 'despesa');
                
                if (temRec && temDes) classeCor = "has-ambos";
                else if (temRec) classeCor = "has-receita";
                else if (temDes) classeCor = "has-despesa";

                grid.innerHTML += `
                    <div class="calendar-day ${classeCor} ${dia === hoje.getDate() ? 'today' : ''}" onclick="focarData('${dataString}')">
                        ${dia}
                    </div>`;
            }
        }

        function importarExcel(event) {
            const file = event.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const linhas = e.target.result.split(/\r?\n/).slice(1);
                    let novos = [];
                    linhas.forEach(l => {
                        const c = l.split(';');
                        if (c.length >= 5) {
                            novos.push({ id: Date.now() + Math.random(), desc: c[0].replace(/"/g, ''), valor: parseFloat(c[1].replace(',', '.')), tipo: c[2].trim(), dataVenc: c[3].trim(), status: c[4].trim() });
                        }
                    });
                    if (novos.length > 0) {
                        transacoes = [...transacoes, ...novos];
                        atualizarApp();
                        alert("Importado com sucesso!");
                    }
                } catch (err) { alert("Erro ao importar."); }
            };
            reader.readAsText(file);
        }

        function exportarExcel() {
            if (transacoes.length === 0) return alert("Sem dados.");
            let csv = "\uFEFFDescricao;Valor;Tipo;Vencimento;Status\n";
            transacoes.forEach(t => csv += `${t.desc};${t.valor.toFixed(2).replace('.', ',')};${t.tipo};${t.dataVenc};${t.status}\n`);
            const link = document.createElement("a");
            link.href = URL.createObjectURL(new Blob([csv], { type: 'text/csv' }));
            link.download = "financeiro.csv";
            link.click();
        }

        function atualizarApp() {
            localStorage.setItem('finance_app_v4', JSON.stringify(transacoes));
            const lista = document.getElementById('lista');
            let r = 0, d = 0;
            lista.innerHTML = '';
            transacoes.sort((a, b) => new Date(b.dataVenc) - new Date(a.dataVenc));
            transacoes.forEach(t => {
                if (t.tipo === 'receita') r += t.valor; else d += t.valor;
                lista.innerHTML += `
                    <div class="card flex justify-between items-center border-l-8 ${t.tipo === 'receita' ? 'border-green-500' : 'border-red-500'} mb-3">
                        <div>
                            <p class="font-bold text-gray-800 text-xs uppercase">${t.desc}</p>
                            <p class="text-[10px] text-gray-400">${t.dataVenc.split('-').reverse().join('/')} | ${t.status.toUpperCase()}</p>
                            <div class="mt-1 flex gap-3">
                                <button onclick="editar(${t.id})" class="text-blue-600 text-[10px] font-bold">EDITAR</button>
                                <button onclick="remover(${t.id})" class="text-gray-400 text-[10px]">EXCLUIR</button>
                            </div>
                        </div>
                        <p class="font-black text-sm ${t.tipo === 'receita' ? 'text-green-600' : 'text-red-600'}">R$ ${t.valor.toFixed(2)}</p>
                    </div>`;
            });
            document.getElementById('total-receitas').innerText = `R$ ${r.toFixed(2)}`;
            document.getElementById('total-despesas').innerText = `R$ ${d.toFixed(2)}`;
            gerarCalendario();
        }

        window.onload = () => {
            atualizarApp();
            const hoje = new Date().toISOString().split('T')[0];
            const p = transacoes.filter(t => t.status === 'pendente' && t.tipo === 'despesa' && t.dataVenc >= hoje).sort((a,b)=>new Date(a.dataVenc)-new Date(b.dataVenc))[0];
            if (p) alert(`🔔 CONTA PRÓXIMA: "${p.desc.toUpperCase()}" vence em ${p.dataVenc.split('-').reverse().join('/')}`);
        };
    </script>
</body>
</html>


