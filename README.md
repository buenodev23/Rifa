GitHub Copilot Chat Assistant

<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Rifa Digital — Pagamento via mylnker</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body { box-sizing: border-box; background: #0b1226; color: #eaeaea; font-family: Inter, system-ui, -apple-system, sans-serif; }
    .numero-item { transition: transform .12s ease, opacity .12s ease; cursor: pointer; user-select: none; }
    .numero-item:hover { transform: scale(1.06); }
    .numero-item.ocupado { opacity: .35; cursor: default; transform: none; }
    .numero-item.pendente { outline: 3px solid rgba(233,69,96,0.14); transform: scale(1.08); }
    .numero-item.confirmado { background: #0b7a53 !important; }
    .modal-backdrop { background: rgba(0,0,0,0.7); backdrop-filter: blur(4px); }
    .toast { animation: slideIn .25s ease-out; transition: opacity .3s ease, transform .3s ease; }
    @keyframes slideIn { from { transform: translateY(-6%); opacity: 0 } to { transform: translateY(0); opacity: 1 } }
  </style>
</head>
<body>
  <div class="w-full min-h-screen p-4 md:p-8">
    <div id="header" class="w-full max-w-6xl mx-auto rounded-2xl p-6 md:p-8 mb-6 shadow-2xl bg-[#16213e] relative">
      <h1 class="text-center font-bold mb-2 text-3xl">Grande Rifa Beneficente</h1>
      <p class="text-center font-medium mb-4 text-xl">🎁 iPhone 13 Mini — Pagamento via mylnker</p>

      <div class="absolute right-6 top-6">
        <button id="btn-admin-open" class="px-4 py-2 rounded-xl bg-[#0f3460] font-semibold">Admin</button>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="info-item rounded-xl p-4 text-center bg-[#16213e]">
          <div class="info-label font-medium mb-2 text-sm">Preço por Número</div>
          <div id="preco-value" class="info-value font-bold text-lg">R$ 10,00</div>
        </div>
        <div class="info-item rounded-xl p-4 text-center bg-[#16213e]">
          <div class="info-label font-medium mb-2 text-sm">Data do Sorteio</div>
          <div id="data-value" class="info-value font-bold text-lg">25/12/2024</div>
        </div>
        <div class="info-item rounded-xl p-4 text-center bg-[#16213e]">
          <div class="info-label font-medium mb-2 text-sm">Números Disponíveis</div>
          <div id="disponivel-value" class="info-value font-bold text-lg">1000</div>
        </div>
      </div>
    </div>

    <div id="numeros-grid" class="grid grid-cols-5 sm:grid-cols-8 md:grid-cols-10 lg:grid-cols-12 xl:grid-cols-15 gap-2 max-w-6xl mx-auto"></div>

    <div class="max-w-6xl mx-auto mt-6 text-sm text-gray-300">
      <p><strong>Fluxo:</strong> Você seleciona um número e preenche nome/WhatsApp. Ao confirmar será criado um pedido temporário (pendente) e você será redirecionado ao mylnker para concluir o pagamento. Se o pagamento não for efetuado em até 20 minutos, o número volta a ficar disponível automaticamente. Use o painel Admin (senha embutida) para confirmar pagamentos manualmente.</p>
    </div>
  </div>

  <div id="toast-container" class="fixed top-4 right-4 z-50"></div>

  <!-- Modal de reserva -->
  <div id="modal" class="fixed inset-0 z-40 hidden items-center justify-center p-4" aria-hidden="true" role="dialog" aria-modal="true">
    <div class="modal-backdrop absolute inset-0" onclick="fecharModal()" aria-hidden="true"></div>
    <div class="relative w-full max-w-md rounded-2xl p-6 shadow-2xl bg-[#16213e]">
      <h2 class="font-bold mb-4 text-center text-xl">Reservar Número</h2>
      <p id="modal-numero" class="text-center mb-4 font-bold text-2xl"></p>
      <form id="form-reserva" class="space-y-4" onsubmit="event.preventDefault(); confirmarReserva()">
        <div>
          <label class="block mb-2 font-medium">Seu Nome</label>
          <input type="text" id="input-nome" required class="w-full px-4 py-3 rounded-lg border-2 outline-none bg-[#0b1226] text-[#eaeaea] border-[#0f3460]" placeholder="Digite seu nome completo" />
        </div>
        <div>
          <label class="block mb-2 font-medium">WhatsApp</label>
          <input type="tel" id="input-telefone" required class="w-full px-4 py-3 rounded-lg border-2 outline-none bg-[#0b1226] text-[#eaeaea] border-[#0f3460]" placeholder="(00) 00000-0000" />
          <p class="text-xs text-gray-400 mt-1">Formato esperado: somente números ou (00) 00000-0000; exemplo: 5591999887766 ou (91) 99988-7766</p>
        </div>

        <div class="flex gap-3">
          <button type="button" onclick="fecharModal()" class="flex-1 px-6 py-3 rounded-lg font-bold bg-[#0f3460] hover:opacity-90">Cancelar</button>
          <button type="submit" id="btn-confirmar" class="flex-1 px-6 py-3 rounded-lg font-bold bg-[#e94560] hover:opacity-90">Confirmar e Pagar</button>
        </div>
      </form>
    </div>
  </div>

  <!-- Admin: login -->
  <div id="modal-admin-login" class="fixed inset-0 z-50 hidden items-center justify-center p-4" aria-hidden="true">
    <div class="modal-backdrop absolute inset-0" onclick="fecharAdmin()" aria-hidden="true"></div>
    <div class="relative w-full max-w-md rounded-2xl p-6 shadow-2xl bg-[#16213e]">
      <h2 class="font-bold mb-4 text-center text-xl">Admin — Login</h2>
      <form id="form-admin-login" onsubmit="event.preventDefault(); adminLogin()">
        <div class="mb-4">
          <label class="block mb-2 font-medium">Senha</label>
          <input id="admin-password-input" type="password" required class="w-full px-4 py-3 rounded-lg bg-[#0b1226] border-2 border-[#0f3460] text-[#eaeaea]" placeholder="Digite a senha do admin" />
        </div>
        <div class="flex gap-3">
          <button type="button" onclick="fecharAdmin()" class="flex-1 px-6 py-3 rounded-lg font-bold bg-[#0f3460]">Cancelar</button>
          <button type="submit" class="flex-1 px-6 py-3 rounded-lg font-bold bg-[#e94560]">Entrar</button>
        </div>
      </form>
    </div>
  </div>

  <!-- Admin: painel -->
  <div id="modal-admin" class="fixed inset-0 z-50 hidden items-center justify-center p-4" aria-hidden="true" role="dialog" aria-modal="true">
    <div class="modal-backdrop absolute inset-0" onclick="fecharAdminPanel()" aria-hidden="true"></div>
    <div class="relative w-full max-w-4xl rounded-2xl p-6 shadow-2xl bg-[#16213e] max-h-[85vh] overflow-auto">
      <div class="flex items-center justify-between mb-4">
        <h2 class="font-bold text-xl">Painel Admin</h2>
        <div class="flex items-center gap-3">
          <button onclick="exportState()" class="px-3 py-2 rounded bg-[#0f3460]">Exportar JSON</button>
          <button onclick="fecharAdminPanel()" class="px-3 py-2 rounded bg-[#e94560]">Fechar</button>
        </div>
      </div>

      <div class="mb-6">
        <h3 class="font-semibold mb-2">Pedidos Pendentes</h3>
        <div id="admin-pendentes" class="space-y-2"></div>
      </div>

      <div class="mb-6">
        <h3 class="font-semibold mb-2">Pedidos Confirmados</h3>
        <div id="admin-confirmados" class="space-y-2"></div>
      </div>

      <div>
        <h3 class="font-semibold mb-2">Ações</h3>
        <div class="flex gap-2">
          <button onclick="liberarTodosExpirados()" class="px-3 py-2 rounded bg-[#0f3460]">Liberar expirados agora</button>
          <button onclick="limparConfirmados()" class="px-3 py-2 rounded bg-[#ef4444]">Limpar confirmados</button>
        </div>
      </div>
    </div>
  </div>

<script>
/*
  Arquivo standalone (sem backend)
  - TTL: 20 minutos (configurado abaixo)
  - Admin password embutida: "2302"
  - Redireciona para https://mylnker.com/duck-duck?message=...
  - Persistência: localStorage (chave: 'rifa_state_v1')
*/

const CONFIG = {
  TOTAL_NUMEROS: 1000,
  TTL_MINUTES: 20, // valor definido por você
  ADMIN_PASSWORD: '2302',
  STORAGE_KEY: 'rifa_state_v1',
  PRICE: 'R$ 10,00',
  MYLNKER_URL: 'https://mylnker.com/duck-duck?message='
};

document.getElementById('preco-value').textContent = CONFIG.PRICE;

let state = {
  numbers: {}, // numero -> { status: 'available'|'pending'|'confirmed', orderId?: string }
  orders: {}   // orderId -> { id, numero, nome, telefone, status: 'pending'|'confirmed'|'failed', createdAt, expiresAt }
};

// --- util ---
function uuidv4() {
  // simple UUID (not cryptographically secure) sufficient here
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
    const r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}
function nowMs(){ return Date.now(); }
function saveState(){ localStorage.setItem(CONFIG.STORAGE_KEY, JSON.stringify(state)); }
function loadState(){
  const raw = localStorage.getItem(CONFIG.STORAGE_KEY);
  if(raw){
    try { state = JSON.parse(raw); } catch(e){ console.error('Erro ao parsear estado', e); }
  } else {
    // inicializa numbers como available
    for(let i=1;i<=CONFIG.TOTAL_NUMEROS;i++){
      const numero = String(i).padStart(4,'0');
      state.numbers[numero] = { status: 'available' };
    }
    saveState();
  }
}
function formatTime(ms){ const d = new Date(ms); return d.toLocaleString(); }

// --- renderização ---
const grid = document.getElementById('numeros-grid');
const disponivelEl = document.getElementById('disponivel-value');
let numeroSelecionado = null;

function renderNumeros(){
  grid.innerHTML = '';
  const frag = document.createDocumentFragment();
  for(let i=1;i<=CONFIG.TOTAL_NUMEROS;i++){
    const numero = String(i).padStart(4,'0');
    const data = state.numbers[numero] || { status: 'available' };
    const div = document.createElement('div');
    div.className = 'numero-item flex items-center justify-center rounded-lg font-bold aspect-square';
    div.style.background = '#0f3460';
    div.textContent = numero;
    div.dataset.num = numero;

    if(data.status === 'available'){
      div.onclick = () => abrirModal(numero);
      div.setAttribute('role','button');
      div.setAttribute('aria-label', `Número ${numero} disponível`);
    } else if(data.status === 'pending'){
      div.classList.add('pendente');
      div.onclick = null;
      div.setAttribute('aria-label', `Número ${numero} pendente`);
    } else if(data.status === 'confirmed'){
      div.classList.add('confirmado');
      div.onclick = null;
      div.setAttribute('aria-label', `Número ${numero} confirmado`);
    }

    frag.appendChild(div);
  }
  grid.appendChild(frag);
  atualizarDisponiveis();
}

function atualizarDisponiveis(){
  let count = 0;
  for(const num in state.numbers) if(state.numbers[num].status === 'available') count++;
  disponivelEl.textContent = count;
}

// --- modal reserve ---
const modal = document.getElementById('modal');
const modalNumeroEl = document.getElementById('modal-numero');
const inputNome = document.getElementById('input-nome');
const inputTelefone = document.getElementById('input-telefone');

function abrirModal(numero){
  numeroSelecionado = numero;
  modalNumeroEl.textContent = 'Número ' + numero;
  modal.style.display = 'flex';
  modal.setAttribute('aria-hidden','false');
  inputNome.value = '';
  inputTelefone.value = '';
  setTimeout(()=> inputNome.focus(), 80);
  // visual pendente local while user fills (not yet persisted)
  const el = document.querySelector(`[data-num="${numero}"]`);
  if(el) el.classList.add('pendente');
}

function fecharModal(){
  modal.style.display = 'none';
  modal.setAttribute('aria-hidden','true');
  if(numeroSelecionado){
    const el = document.querySelector(`[data-num="${numeroSelecionado}"]`);
    if(el && (!state.numbers[numeroSelecionado] || state.numbers[numeroSelecionado].status === 'available')){
      el.classList.remove('pendente');
    }
  }
  numeroSelecionado = null;
}

// --- validação telefone simples ---
function validarTelefone(tel){
  if(!tel) return false;
  const onlyDigits = tel.replace(/\D/g,'');
  return onlyDigits.length >= 10 && onlyDigits.length <= 13;
}

// --- criar pedido e redirecionar ao mylnker ---
function confirmarReserva(){
  const nome = inputNome.value.trim();
  const telefone = inputTelefone.value.trim();
  if(!nome || !telefone){ showToast('Preencha todos os campos','error'); return; }
  if(!validarTelefone(telefone)){ showToast('Telefone inválido','error'); return; }
  if(!numeroSelecionado){ showToast('Nenhum número selecionado','error'); return; }

  // verifica disponibilidade atual
  const current = state.numbers[numeroSelecionado];
  if(current && current.status !== 'available'){
    showToast('Número não está mais disponível','error');
    fecharModal();
    renderNumeros();
    return;
  }

  const orderId = uuidv4();
  const createdAt = nowMs();
  const expiresAt = createdAt + CONFIG.TTL_MINUTES * 60 * 1000;

  // marca pending
  state.numbers[numeroSelecionado] = { status: 'pending', orderId };
  state.orders[orderId] = {
    id: orderId,
    numero: numeroSelecionado,
    nome,
    telefone,
    status: 'pending',
    createdAt,
    expiresAt
  };
  saveState();
  renderNumeros();
  fecharModal();

  showToast('Pedido criado. Você será redirecionado para o pagamento (mylnker).','success');

  // monta mensagem para mylnker (inclui orderId para identificação manual depois)
  const mensagem = `Reserva da rifa\nNúmero: ${numeroSelecionado}\nNome: ${nome}\nWhatsApp: ${telefone}\nOrderId: ${orderId}\nValor: ${CONFIG.PRICE}\n-- Por favor, faça o pagamento e avise no admin para confirmar.`;
  const url = CONFIG.MYLNKER_URL + encodeURIComponent(mensagem);

  // abre em nova aba (padrão)
  window.open(url, '_blank', 'noopener,noreferrer');
}

// --- toasts ---
const toastContainer = document.getElementById('toast-container');
function showToast(message, type='info'){
  const toast = document.createElement('div');
  toast.className = 'toast px-6 py-3 rounded-lg shadow-lg mb-2';
  toast.style.background = type === 'success' ? '#0ea5a4' : (type === 'error' ? '#ef4444' : '#334155');
  toast.style.color = '#f8fafc';
  toast.textContent = message;
  toastContainer.appendChild(toast);
  setTimeout(()=> {
    toast.style.opacity = '0';
    toast.style.transform = 'translateY(-6px)';
    setTimeout(()=> toast.remove(), 350);
  }, 3000);
}

// --- expirador automático ---
function liberarExpirados(){
  const now = nowMs();
  let changed = false;
  for(const id in state.orders){
    const order = state.orders[id];
    if(order.status === 'pending' && order.expiresAt <= now){
      // liberar
      order.status = 'failed';
      const num = order.numero;
      if(state.numbers[num] && state.numbers[num].orderId === id){
        state.numbers[num] = { status: 'available' };
      }
      changed = true;
    }
  }
  if(changed){ saveState(); renderNumeros(); }
}

// periodic check every 30s
setInterval(liberarExpirados, 30 * 1000);

// --- admin ---
const ADMIN_PASSWORD = CONFIG.ADMIN_PASSWORD;
const adminLoginModal = document.getElementById('modal-admin-login');
const adminPanelModal = document.getElementById('modal-admin');

document.getElementById('btn-admin-open').addEventListener('click', ()=> {
  adminLoginModal.style.display = 'flex';
  adminLoginModal.setAttribute('aria-hidden','false');
  document.getElementById('admin-password-input').value = '';
  setTimeout(()=> document.getElementById('admin-password-input').focus(), 80);
});

function fecharAdmin(){
  adminLoginModal.style.display = 'none';
  adminLoginModal.setAttribute('aria-hidden','true');
}
function fecharAdminPanel(){
  adminPanelModal.style.display = 'none';
  adminPanelModal.setAttribute('aria-hidden','true');
}

function adminLogin(){
  const p = document.getElementById('admin-password-input').value;
  if(p === ADMIN_PASSWORD){
    fecharAdmin();
    openAdminPanel();
  } else {
    showToast('Senha incorreta','error');
  }
}

function openAdminPanel(){
  renderAdminLists();
  adminPanelModal.style.display = 'flex';
  adminPanelModal.setAttribute('aria-hidden','false');
}

function renderAdminLists(){
  const pendentesEl = document.getElementById('admin-pendentes');
  const confirmadosEl = document.getElementById('admin-confirmados');
  pendentesEl.innerHTML = '';
  confirmadosEl.innerHTML = '';

  const orders = Object.values(state.orders).sort((a,b)=> b.createdAt - a.createdAt);
  for(const order of orders){
    const el = document.createElement('div');
    el.className = 'p-3 rounded bg-[#0b1226] flex items-center justify-between';
    const left = document.createElement('div');
    left.innerHTML = `<div class="font-semibold">#${order.numero} — ${order.nome}</div>
                      <div class="text-xs text-gray-400">Tel: ${order.telefone} • Criado: ${formatTime(order.createdAt)} • Expira: ${formatTime(order.expiresAt)}</div>
                      <div class="text-xs text-gray-400">OrderId: ${order.id}</div>`;
    const right = document.createElement('div');
    right.className = 'flex gap-2';

    if(order.status === 'pending'){
      const btnConfirm = document.createElement('button');
      btnConfirm.className = 'px-3 py-1 rounded bg-[#0f3460]';
      btnConfirm.textContent = 'Confirmar (Pago)';
      btnConfirm.onclick = ()=> adminConfirm(order.id);
      const btnRelease = document.createElement('button');
      btnRelease.className = 'px-3 py-1 rounded bg-[#ef4444]';
      btnRelease.textContent = 'Liberar';
      btnRelease.onclick = ()=> adminRelease(order.id);
      right.appendChild(btnConfirm);
      right.appendChild(btnRelease);
      el.appendChild(left); el.appendChild(right);
      pendentesEl.appendChild(el);
    } else if(order.status === 'confirmed'){
      const btnRelease = document.createElement('button');
      btnRelease.className = 'px-3 py-1 rounded bg-[#ef4444]';
      btnRelease.textContent = 'Liberar';
      btnRelease.onclick = ()=> adminRelease(order.id);
      right.appendChild(btnRelease);
      el.appendChild(left); el.appendChild(right);
      confirmadosEl.appendChild(el);
    } else {
      // failed/others -> show in pendentes maybe
      const small = document.createElement('div');
      small.className = 'text-xs text-gray-400';
      small.textContent = 'Status: ' + order.status;
      left.appendChild(small);
      const btnClear = document.createElement('button');
      btnClear.className = 'px-3 py-1 rounded bg-[#0f3460]';
      btnClear.textContent = 'Remover';
      btnClear.onclick = ()=> { delete state.orders[order.id]; saveState(); renderAdminLists(); renderNumeros(); };
      right.appendChild(btnClear);
      el.appendChild(left); el.appendChild(right);
      pendentesEl.appendChild(el);
    }
  }
}

function adminConfirm(orderId){
  const order = state.orders[orderId];
  if(!order) return showToast('Pedido não encontrado','error');
  order.status = 'confirmed';
  const num = order.numero;
  state.numbers[num] = { status: 'confirmed', orderId };
  saveState();
  renderAdminLists();
  renderNumeros();
  showToast('Pedido confirmado','success');
}

function adminRelease(orderId){
  const order = state.orders[orderId];
  if(!order) return showToast('Pedido não encontrado','error');
  const num = order.numero;
  // liberar numero
  state.numbers[num] = { status: 'available' };
  delete state.orders[orderId];
  saveState();
  renderAdminLists();
  renderNumeros();
  showToast('Pedido liberado','success');
}

function exportState(){
  const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(state, null, 2));
  const a = document.createElement('a');
  a.href = dataStr;
  a.download = 'rifa_state_export.json';
  document.body.appendChild(a);
  a.click();
  a.remove();
}

function liberarTodosExpirados(){
  liberarExpirados();
  renderAdminLists();
  showToast('Expirados verificados e liberados se houver','success');
}
function limparConfirmados(){
  // cuidado: vai remover todas as confirmações (opcional)
  if(!confirm('Remover todos os pedidos confirmados? Isso deixará os números disponíveis de novo.')) return;
  for(const id in state.orders){
    if(state.orders[id].status === 'confirmed'){
      const num = state.orders[id].numero;
      state.numbers[num] = { status: 'available' };
      delete state.orders[id];
    }
  }
  saveState();
  renderAdminLists();
  renderNumeros();
  showToast('Confirmados removidos','success');
}

// --- inicialização ---
loadState();
renderNumeros();

// checar expirados imediatamente
liberarExpirados();

// fechar modals com ESC
document.addEventListener('keydown', (e) => {
  if(e.key === 'Escape'){
    if(modal.style.display === 'flex') fecharModal();
    if(adminLoginModal.style.display === 'flex') fecharAdmin();
    if(adminPanelModal.style.display === 'flex') fecharAdminPanel();
  }
});

// Fornece uma pequena interface para o usuário ver seus pedidos locais (opcional)
(function renderUserOrdersWidget(){
  const container = document.createElement('div');
  container.className = 'fixed left-4 bottom-4 z-40';
  container.innerHTML = `<div id="my-orders" class="p-3 rounded-lg bg-[#0b1226] text-sm shadow-lg max-h-72 overflow-auto"></div>`;
  document.body.appendChild(container);
  updateUserOrdersList();
  setInterval(updateUserOrdersList, 5000);

  function updateUserOrdersList(){
    const el = document.getElementById('my-orders');
    el.innerHTML = '<div class="font-semibold mb-2">Meus Pedidos Locais</div>';
    const myOrders = Object.values(state.orders).sort((a,b)=> b.createdAt - a.createdAt);
    if(myOrders.length === 0){ el.innerHTML += '<div class="text-gray-400">Nenhum pedido local.</div>'; return; }
    for(const o of myOrders){
      const div = document.createElement('div');
      div.className = 'mb-2 p-2 rounded bg-[#081025]';
      div.innerHTML = `<div class="font-semibold">#${o.numero} — ${o.nome} <span class="text-xs text-gray-400">(${o.status})</span></div>
                       <div class="text-xs text-gray-400">OrderId: ${o.id}</div>
                       <div class="text-xs text-gray-400">Criado: ${formatTime(o.createdAt)}</div>
                       ${o.status === 'pending' ? `<div class="text-xs text-gray-300">Expira: ${formatTime(o.expiresAt)}</div>` : ''}`;
      el.appendChild(div);
    }
  }
})();

</script>
</body>
</html>
