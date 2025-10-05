
// =================================================================
// 📖 BÍBLIA ONLINE COM LOGIN REAL + VERIFICAÇÃO POR E-MAIL (GMAIL)
// Autor: Assistente
// Este script gera automaticamente todos os arquivos do projeto.
// =================================================================

const fs = require('fs');
const path = require('path');
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const bcrypt = require('bcrypt');
const session = require('express-session');
const csrf = require('csurf');
const nodemailer = require('nodemailer');

// Verifica se o .env existe; se não, cria um modelo
const envPath = path.join(__dirname, '.env');
if (!fs.existsSync(envPath)) {
  const envExample = `GMAIL_USER=sagradab012@gmail.com
GMAIL_APP_PASSWORD=sua_senha_de_app_de_16_digitos_aqui
SESSION_SECRET=sua_chave_secreta_muito_forte_aqui_123!
NODE_ENV=development
PORT=3000`;
  fs.writeFileSync(envPath, envExample);
  console.log('✅ Arquivo .env criado! Configure suas credenciais do Gmail.');
}

require('dotenv').config();

// Cria pastas se não existirem
const dirs = ['public', 'utils'];
dirs.forEach(dir => {
  if (!fs.existsSync(dir)) fs.mkdirSync(dir);
});

// =================================================================
// GERAÇÃO AUTOMÁTICA DE ARQUIVOS
// =================================================================

// utils/cpfValidator.js
const cpfValidatorContent = `function validarCPF(cpf) {
  cpf = cpf.replace(/[^\d]/g, '');
  if (cpf.length !== 11) return false;
  if (/^(\d)\1{10}$/.test(cpf)) return false;

  let soma = 0;
  for (let i = 0; i < 9; i++) {
    soma += parseInt(cpf.charAt(i)) * (10 - i);
  }
  let resto = 11 - (soma % 11);
  let digito1 = resto > 9 ? 0 : resto;

  soma = 0;
  for (let i = 0; i < 10; i++) {
    soma += parseInt(cpf.charAt(i)) * (11 - i);
  }
  resto = 11 - (soma % 11);
  let digito2 = resto > 9 ? 0 : resto;

  return digito1 === parseInt(cpf.charAt(9)) && digito2 === parseInt(cpf.charAt(10));
}

module.exports = { validarCPF };`;

fs.writeFileSync(path.join(__dirname, 'utils', 'cpfValidator.js'), cpfValidatorContent);

// public/style.css
const styleContent = `body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f5f7fa;
  margin: 0;
  padding: 0;
}
header {
  background: #2c3e50;
  color: white;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
header h1 { margin: 0; font-size: 1.5rem; }
.search-box {
  text-align: center;
  margin: 2rem;
}
.search-box select, .search-box button {
  padding: 0.8rem;
  font-size: 1rem;
  margin: 0 5px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
.search-box button {
  background: #3498db;
  color: white;
  border: none;
  cursor: pointer;
}
.search-box button:hover { background: #2980b9; }
#result {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}
.reference { font-weight: bold; color: #2c3e50; font-size: 1.3rem; margin-bottom: 1rem; }
.text { font-size: 1.2rem; line-height: 1.6; }
.historico-secao {
  max-width: 800px;
  margin: 2rem auto;
  padding: 1rem 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}
.historico-secao h3 { margin-top: 0; color: #2c3e50; }
.historico-item { padding: 0.8rem; border-bottom: 1px solid #eee; }
.historico-item:last-child { border-bottom: none; }
.historico-ref { font-weight: bold; color: #2980b9; }
.historico-data { font-size: 0.85rem; color: #7f8c8d; margin-top: 4px; }
footer {
  text-align: center;
  margin-top: 3rem;
  padding: 1rem;
  color: #7f8c8d;
  font-size: 0.9rem;
}
.auth-container {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0,0,0,0.1);
  width: 350px;
  text-align: center;
  margin: 2rem auto;
}
.auth-container h2 { margin-top: 0; color: #2c3e50; }
.auth-container input {
  width: 100%;
  padding: 0.8rem;
  margin: 0.8rem 0;
  border: 1px solid #ddd;
  border-radius: 6px;
  box-sizing: border-box;
}
.auth-container button {
  width: 100%;
  padding: 0.9rem;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  cursor: pointer;
  margin-top: 1rem;
}
.auth-container button:hover { background: #2980b9; }
.auth-container a {
  color: #3498db;
  text-decoration: none;
  display: inline-block;
  margin-top: 1rem;
}
.auth-container a:hover { text-decoration: underline; }`;

fs.writeFileSync(path.join(__dirname, 'public', 'style.css'), styleContent);

// Livros da Bíblia (para o script.js)
const livrosArray = `const livros = [
  { nome: "Gênesis", abrev: "Gn", cap: 50 },
  { nome: "Êxodo", abrev: "Êx", cap: 40 },
  { nome: "Levítico", abrev: "Lv", cap: 27 },
  { nome: "Números", abrev: "Nm", cap: 36 },
  { nome: "Deuteronômio", abrev: "Dt", cap: 34 },
  { nome: "Josué", abrev: "Js", cap: 24 },
  { nome: "Juízes", abrev: "Jz", cap: 21 },
  { nome: "Rute", abrev: "Rt", cap: 4 },
  { nome: "1 Samuel", abrev: "1Sm", cap: 31 },
  { nome: "2 Samuel", abrev: "2Sm", cap: 24 },
  { nome: "1 Reis", abrev: "1Rs", cap: 22 },
  { nome: "2 Reis", abrev: "2Rs", cap: 25 },
  { nome: "1 Crônicas", abrev: "1Cr", cap: 29 },
  { nome: "2 Crônicas", abrev: "2Cr", cap: 36 },
  { nome: "Esdras", abrev: "Ed", cap: 10 },
  { nome: "Neemias", abrev: "Ne", cap: 13 },
  { nome: "Ester", abrev: "Et", cap: 10 },
  { nome: "Jó", abrev: "Jó", cap: 42 },
  { nome: "Salmos", abrev: "Sl", cap: 150 },
  { nome: "Provérbios", abrev: "Pv", cap: 31 },
  { nome: "Eclesiastes", abrev: "Ec", cap: 12 },
  { nome: "Cantares", abrev: "Ct", cap: 8 },
  { nome: "Isaías", abrev: "Is", cap: 66 },
  { nome: "Jeremias", abrev: "Jr", cap: 52 },
  { nome: "Lamentações", abrev: "Lm", cap: 5 },
  { nome: "Ezequiel", abrev: " Ez", cap: 48 },
  { nome: "Daniel", abrev: "Dn", cap: 12 },
  { nome: "Oséias", abrev: "Os", cap: 14 },
  { nome: "Joel", abrev: "Jl", cap: 3 },
  { nome: "Amós", abrev: "Am", cap: 9 },
  { nome: "Obadias", abrev: "Ob", cap: 1 },
  { nome: "Jonas", abrev: "Jn", cap: 4 },
  { nome: "Miqueias", abrev: "Mq", cap: 7 },
  { nome: "Naum", abrev: "Na", cap: 3 },
  { nome: "Habacuque", abrev: "Hb", cap: 3 },
  { nome: "Sofonias", abrev: "So", cap: 3 },
  { nome: "Ageu", abrev: "Ag", cap: 2 },
  { nome: "Zacarias", abrev: "Zc", cap: 14 },
  { nome: "Malaquias", abrev: "Ml", cap: 4 },
  { nome: "Mateus", abrev: "Mt", cap: 28 },
  { nome: "Marcos", abrev: "Mc", cap: 16 },
  { nome: "Lucas", abrev: "Lc", cap: 24 },
  { nome: "João", abrev: "Jo", cap: 21 },
  { nome: "Atos", abrev: "At", cap: 28 },
  { nome: "Romanos", abrev: "Rm", cap: 16 },
  { nome: "1 Coríntios", abrev: "1Co", cap: 16 },
  { nome: "2 Coríntios", abrev: "2Co", cap: 13 },
  { nome: "Gálatas", abrev: "Gl", cap: 6 },
  { nome: "Efésios", abrev: "Ef", cap: 6 },
  { nome: "Filipenses", abrev: "Fp", cap: 4 },
  { nome: "Colossenses", abrev: "Cl", cap: 4 },
  { nome: "1 Tessalonicenses", abrev: "1Ts", cap: 5 },
  { nome: "2 Tessalonicenses", abrev: "2Ts", cap: 3 },
  { nome: "1 Timóteo", abrev: "1Tm", cap: 6 },
  { nome: "2 Timóteo", abrev: "2Tm", cap: 4 },
  { nome: "Tito", abrev: "Tt", cap: 3 },
  { nome: "Filemon", abrev: "Fm", cap: 1 },
  { nome: "Hebreus", abrev: "Hb", cap: 13 },
  { nome: "Tiago", abrev: "Tg", cap: 5 },
  { nome: "1 Pedro", abrev: "1Pe", cap: 5 },
  { nome: "2 Pedro", abrev: "2Pe", cap: 3 },
  { nome: "1 João", abrev: "1Jo", cap: 5 },
  { nome: "2 João", abrev: "2Jo", cap: 1 },
  { nome: "3 João", abrev: "3Jo", cap: 1 },
  { nome: "Judas", abrev: "Jd", cap: 1 },
  { nome: "Apocalipse", abrev: "Ap", cap: 22 }
];`;

// public/script.js
const scriptContent = livrosArray + `

// Preenche o select de livros
function carregarLivros() {
  const select = document.getElementById('livroSelect');
  livros.forEach(livro => {
    const opt = document.createElement('option');
    opt.value = livro.abrev;
    opt.textContent = \`\${livro.nome} (\${livro.cap} cap.)\`;
    select.appendChild(opt);
  });
}

document.getElementById('livroSelect').addEventListener('change', function() {
  const capSelect = document.getElementById('capituloSelect');
  capSelect.innerHTML = '<option value="">Capítulo</option>';
  const livroAbrev = this.value;
  const livro = livros.find(l => l.abrev === livroAbrev);
  if (!livro) return;

  for (let i = 1; i <= livro.cap; i++) {
    const opt = document.createElement('option');
    opt.value = i;
    opt.textContent = i;
    capSelect.appendChild(opt);
  }
});

document.getElementById('buscarBtn').addEventListener('click', function() {
  const livro = document.getElementById('livroSelect').value;
  const capitulo = document.getElementById('capituloSelect').value;
  if (!livro || !capitulo) {
    alert('Selecione livro e capítulo.');
    return;
  }
  const referencia = \`\${livro} \${capitulo}\`;
  document.getElementById('reference').value = referencia;
  fetchVerse();
});

async function fetchVerse() {
  const input = document.getElementById('reference').value.trim();
  const resultDiv = document.getElementById('result');
  if (!input) {
    resultDiv.innerHTML = '<p>Digite uma referência bíblica.</p>';
    return;
  }
  resultDiv.innerHTML = '<p>Buscando...</p>';
  try {
    const response = await fetch(\`/api/bible?reference=\${encodeURIComponent(input)}\`);
    const data = await response.json();
    if (data.error) {
      resultDiv.innerHTML = \`<p style="color: red;">\${data.error}</p>\`;
    } else {
      resultDiv.innerHTML = \`
        <div class="reference">\${data.reference}</div>
        <div class="text">\${data.text}</div>
        <p><em>\${data.translation_name}</em></p>
      \`;
      carregarHistorico();
    }
  } catch (err) {
    resultDiv.innerHTML = '<p style="color: red;">Erro ao carregar o versículo.</p>';
  }
}

async function carregarHistorico() {
  try {
    const res = await fetch('/api/historico');
    const historico = await res.json();
    const container = document.getElementById('historicoLista');
    if (historico.length === 0) {
      container.innerHTML = '<p>Nenhum versículo lido ainda.</p>';
      return;
    }
    container.innerHTML = historico.map(item => \`
      <div class="historico-item">
        <div class="historico-ref">\${item.referencia}</div>
        <div>\${item.texto}</div>
        <div class="historico-data">\${new Date(item.data_leitura).toLocaleString('pt-BR')}</div>
      </div>
    \`).join('');
  } catch (err) {
    console.error('Erro ao carregar histórico:', err);
  }
}

document.getElementById('limparHistorico').addEventListener('click', async () => {
  if (!confirm('Tem certeza que deseja limpar todo o histórico de leitura?')) return;
  const token = document.cookie.split('; ').find(row => row.startsWith('XSRF-TOKEN='))?.split('=')[1];
  try {
    const res = await fetch('/api/historico', {
      method: 'DELETE',
      headers: { 'CSRF-Token': token }
    });
    if (res.ok) carregarHistorico();
    else alert('Erro ao limpar histórico.');
  } catch (err) {
    alert('Erro de rede.');
  }
});

function getCookie(name) {
  const value = \`; \${document.cookie}\`;
  const parts = value.split(\`; \${name}=\`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

// Inicialização
if (document.getElementById('livroSelect')) carregarLivros();
if (document.getElementById('historicoLista')) carregarHistorico();
if (document.getElementById('logoutCsrf')) document.getElementById('logoutCsrf').value = getCookie('XSRF-TOKEN');
if (document.getElementById('csrfToken')) document.getElementById('csrfToken').value = getCookie('XSRF-TOKEN');
`;

fs.writeFileSync(path.join(__dirname, 'public', 'script.js'), scriptContent);

// Função para criar HTML com conteúdo
function createHtml(title, bodyContent, extraHead = '', extraScript = '') {
  return `<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>${title}</title>
  <link rel="stylesheet" href="style.css" />
  ${extraHead}
</head>
<body>
  ${bodyContent}
  <script src="script.js"></script>
  ${extraScript}
</body>
</html>`;
}

// public/index.html
const indexHtml = createHtml(
  'Bíblia Sagrada Online',
  `<header>
    <h1>📖 Bíblia Sagrada Online</h1>
    <p>Almeida Revista e Corrigida</p>
    <form id="logoutForm" action="/logout" method="POST" style="display:inline;">
      <input type="hidden" name="_csrf" id="logoutCsrf" />
      <button type="submit" style="color: white; background: #e74c3c; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer;">
        Sair
      </button>
    </form>
  </header>
  <main>
    <div class="search-box">
      <select id="livroSelect" style="width: 200px; padding: 0.8rem; margin-right: 10px;">
        <option value="">Selecione o livro</option>
      </select>
      <select id="capituloSelect" style="width: 100px; padding: 0.8rem; margin-right: 10px;">
        <option value="">Capítulo</option>
      </select>
      <button id="buscarBtn" style="padding: 0.8rem 1.5rem;">Buscar</button>
    </div>
    <div id="result"></div>
    <div class="historico-secao">
      <h3>📜 Histórico de Leitura</h3>
      <button id="limparHistorico" style="margin-bottom: 10px; padding: 6px 10px; background: #e74c3c; color: white; border: none; border-radius: 4px; cursor: pointer;">Limpar Histórico</button>
      <div id="historicoLista"></div>
    </div>
  </main>
  <footer>
    <p>© 2025 Bíblia Online | Dados da <a href="https://bible-api.com" target="_blank">Bible API</a></p>
  </footer>`,
  '',
  '<script>document.getElementById("reference")?.addEventListener("keypress", e => { if (e.key === "Enter") fetchVerse(); });</script>'
);
fs.writeFileSync(path.join(__dirname, 'public', 'index.html'), indexHtml);

// public/login.html
const loginHtml = createHtml(
  'Login - Bíblia Online',
  `<div class="auth-container">
    <h2>📖 Acesse sua conta</h2>
    <p style="color:#7f8c8d; font-size:0.9rem;">
      Após o login, você receberá um código em <strong>sagradab012@gmail.com</strong> para confirmar o acesso.
    </p>
    <form action="/login" method="POST">
      <input type="text" name="cpf" placeholder="CPF (000.000.000-00)" required maxlength="14" id="cpfInput" />
      <input type="password" name="senha" placeholder="Senha" required />
      <input type="hidden" name="_csrf" id="csrfToken" />
      <button type="submit">Entrar</button>
    </form>
    <p><a href="/register.html">Cadastre-se</a> | <a href="/forgot-password.html">Esqueceu a senha?</a></p>
  </div>`,
  '',
  `<script>
    document.getElementById('cpfInput').addEventListener('input', function(e) {
      let v = e.target.value.replace(/\\D/g, '');
      if (v.length > 11) v = v.slice(0, 11);
      let f = '';
      for (let i = 0; i < v.length; i++) {
        if (i === 3 || i === 6) f += '.';
        if (i === 9) f += '-';
        f += v[i];
      }
      e.target.value = f;
    });
  </script>`
);
fs.writeFileSync(path.join(__dirname, 'public', 'login.html'), loginHtml);

// public/register.html
const registerHtml = createHtml(
  'Cadastro - Bíblia Online',
  `<div class="auth-container">
    <h2>📖 Criar Conta</h2>
    <form action="/register" method="POST">
      <input type="text" name="cpf" placeholder="CPF (000.000.000-00)" required maxlength="14" id="cpfInput" />
      <input type="password" name="senha" placeholder="Senha (mín. 6 caracteres)" required minlength="6" />
      <input type="hidden" name="_csrf" id="csrfToken" />
      <button type="submit">Cadastrar</button>
    </form>
    <p>Já tem conta? <a href="/login.html">Faça login</a></p>
  </div>`,
  '',
  `<script>
    document.getElementById('cpfInput').addEventListener('input', function(e) {
      let v = e.target.value.replace(/\\D/g, '');
      if (v.length > 11) v = v.slice(0, 11);
      let f = '';
      for (let i = 0; i < v.length; i++) {
        if (i === 3 || i === 6) f += '.';
        if (i === 9) f += '-';
        f += v[i];
      }
      e.target.value = f;
    });
  </script>`
);
fs.writeFileSync(path.join(__dirname, 'public', 'register.html'), registerHtml);

// public/forgot-password.html
const forgotHtml = createHtml(
  'Recuperar Senha - Bíblia Online',
  `<div class="auth-container">
    <h2>🔐 Recuperar Senha</h2>
    <form action="/forgot-password" method="POST">
      <input type="text" name="cpf" placeholder="CPF (000.000.000-00)" required maxlength="14" id="cpfInput" />
      <input type="hidden" name="_csrf" id="csrfToken" />
      <button type="submit">Enviar link de recuperação</button>
    </form>
    <p><a href="/login.html">Voltar ao login</a></p>
  </div>`,
  '',
  `<script>
    document.getElementById('cpfInput').addEventListener('input', function(e) {
      let v = e.target.value.replace(/\\D/g, '');
      if (v.length > 11) v = v.slice(0, 11);
      let f = '';
      for (let i = 0; i < v.length; i++) {
        if (i === 3 || i === 6) f += '.';
        if (i === 9) f += '-';
        f += v[i];
      }
      e.target.value = f;
    });
  </script>`
);
fs.writeFileSync(path.join(__dirname, 'public', 'forgot-password.html'), forgotHtml);

// public/reset-password.html
const resetHtml = createHtml(
  'Redefinir Senha - Bíblia Online',
  `<div class="auth-container">
    <h2>🔑 Redefinir Senha</h2>
    <form action="/reset-password" method="POST">
      <input type="hidden" name="token" value="" id="tokenInput" />
      <input type="password" name="nova_senha" placeholder="Nova senha (mín. 6 caracteres)" required minlength="6" />
      <input type="password" name="confirmar_senha" placeholder="Confirmar senha" required minlength="6" />
      <input type="hidden" name="_csrf" id="csrfToken" />
      <button type="submit">Redefinir Senha</button>
    </form>
    <p><a href="/login.html">Voltar ao login</a></p>
  </div>`,
  '',
  `<script>
    const urlParams = new URLSearchParams(window.location.search);
    const token = urlParams.get('token');
    if (token) {
      document.getElementById('tokenInput').value = token;
    } else {
      alert('Token inválido.');
      window.location.href = '/login.html';
    }
  </script>`
);
fs.writeFileSync(path.join(__dirname, 'public', 'reset-password.html'), resetHtml);

// =================================================================
// SERVIDOR EXPRESS
// =================================================================

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));
app.use(session({
  secret: process.env.SESSION_SECRET || 'sua-chave-secreta-muito-segura-123!',
  resave: false,
  saveUninitialized: false,
  cookie: { 
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000
  }
}));

const csrfProtection = csrf({ cookie: false });

app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  next();
});

const db = new sqlite3.Database('./database.sqlite', (err) => {
  if (err) console.error('Erro ao abrir o banco:', err.message);
  else {
    db.run(`CREATE TABLE IF NOT EXISTS usuarios (id INTEGER PRIMARY KEY AUTOINCREMENT, cpf TEXT UNIQUE NOT NULL, senha TEXT NOT NULL)`);
    db.run(`CREATE TABLE IF NOT EXISTS historico_leitura (id INTEGER PRIMARY KEY AUTOINCREMENT, usuario_id INTEGER NOT NULL, referencia TEXT NOT NULL, texto TEXT NOT NULL, data_leitura DATETIME DEFAULT CURRENT_TIMESTAMP, FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE)`);
    db.run(`CREATE TABLE IF NOT EXISTS password_resets (id INTEGER PRIMARY KEY AUTOINCREMENT, usuario_id INTEGER NOT NULL, token TEXT UNIQUE NOT NULL, expires_at DATETIME NOT NULL, FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE)`);
    db.run(`CREATE TABLE IF NOT EXISTS codigos_verificacao (id INTEGER PRIMARY KEY AUTOINCREMENT, usuario_id INTEGER NOT NULL, codigo TEXT NOT NULL, criado_em DATETIME DEFAULT CURRENT_TIMESTAMP, usado BOOLEAN DEFAULT 0, FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE)`);
  }
});

const { validarCPF } = require('./utils/cpfValidator');

const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: process.env.GMAIL_USER,
    pass: process.env.GMAIL_APP_PASSWORD
  }
});

transporter.verify((error) => {
  if (error) console.log('Erro no transporte de e-mail:', error);
  else console.log('✅ Servidor de e-mail pronto!');
});

function requireAuth(req, res, next) {
  if (req.session.userId) return next();
  res.redirect('/login.html');
}

// Rotas públicas
app.get('/', requireAuth, (req, res) => res.sendFile(path.join(__dirname, 'public', 'index.html')));
app.get('/login.html', (req, res) => { res.cookie('XSRF-TOKEN', req.csrfToken(), { httpOnly: false }); res.sendFile(path.join(__dirname, 'public', 'login.html')); });
app.get('/register.html', (req, res) => { res.cookie('XSRF-TOKEN', req.csrfToken(), { httpOnly: false }); res.sendFile(path.join(__dirname, 'public', 'register.html')); });
app.get('/forgot-password.html', (req, res) => { res.cookie('XSRF-TOKEN', req.csrfToken(), { httpOnly: false }); res.sendFile(path.join(__dirname, 'public', 'forgot-password.html')); });

app.get('/reset-password.html', (req, res) => {
  const token = req.query.token;
  if (!token) return res.redirect('/login.html');
  db.get('SELECT * FROM password_resets WHERE token = ? AND expires_at > datetime("now")', [token], (err, row) => {
    if (err || !row) return res.send('<script>alert("Token inválido ou expirado."); window.location.href="/login.html";</script>');
    res.cookie('XSRF-TOKEN', req.csrfToken(), { httpOnly: false });
    res.sendFile(path.join(__dirname, 'public', 'reset-password.html'));
  });
});

// Cadastro
app.post('/register', csrfProtection, async (req, res) => {
  const { cpf, senha } = req.body;
  const cpfLimpo = cpf.replace(/[^\d]/g, '');
  if (!validarCPF(cpfLimpo)) return res.status(400).send('<script>alert("CPF inválido!"); window.history.back();</script>');
  if (senha.length < 6) return res.status(400).send('<script>alert("Senha deve ter pelo menos 6 caracteres."); window.history.back();</script>');
  try {
    const hashedSenha = await bcrypt.hash(senha, 10);
    db.run('INSERT INTO usuarios (cpf, senha) VALUES (?, ?)', [cpfLimpo, hashedSenha], function(err) {
      if (err) return res.status(400).send('<script>alert("CPF já cadastrado!"); window.history.back();</script>');
      res.redirect('/login.html');
    });
  } catch (err) {
    res.status(500).send('Erro interno.');
  }
});

// Login com verificação por e-mail
app.post('/login', csrfProtection, async (req, res) => {
  const { cpf, senha } = req.body;
  const cpfLimpo = cpf.replace(/[^\d]/g, '');
  db.get('SELECT * FROM usuarios WHERE cpf = ?', [cpfLimpo], async (err, usuario) => {
    if (err || !usuario) return res.status(401).send('<script>alert("CPF ou senha incorretos."); window.history.back();</script>');
    const senhaValida = await bcrypt.compare(senha, usuario.senha);
    if (!senhaValida) return res.status(401).send('<script>alert("CPF ou senha incorretos."); window.history.back();</script>');
    const codigo = Math.floor(100000 + Math.random() * 900000).toString();
    db.run('INSERT INTO codigos_verificacao (usuario_id, codigo) VALUES (?, ?)', [usuario.id, codigo], async (err) => {
      if (err) return res.status(500).send('<script>alert("Erro ao gerar código."); window.history.back();</script>');
      try {
        await transporter.sendMail({
          from: process.env.GMAIL_USER,
          to: process.env.GMAIL_USER,
          subject: 'Código de Verificação - Bíblia Online',
          text: \`Seu código de acesso é: \${codigo}\\n\\nEste código expira em 10 minutos.\`,
          html: \`<h2>🔐 Código de Verificação</h2><p>Seu código de acesso à Bíblia Online é:</p><h1 style="color:#3498db;">\${codigo}</h1><p>Este código expira em <strong>10 minutos</strong>.</p>\`
        });
        res.send(\`
          <div style="font-family:Arial; padding:2rem; max-width:500px; margin:0 auto; text-align:center;">
            <h2>📧 Verifique seu e-mail</h2>
            <p>Enviamos um código de 6 dígitos para:</p>
            <p style="font-weight:bold; color:#2c3e50;">\${process.env.GMAIL_USER}</p>
            <form action="/verificar-codigo" method="POST" style="margin-top:20px;">
              <input type="hidden" name="usuario_id" value="\${usuario.id}" />
              <input type="text" name="codigo" placeholder="Digite o código" maxlength="6" required
                style="padding:10px; font-size:18px; width:200px; text-align:center;" />
              <input type="hidden" name="_csrf" value="\${req.csrfToken()}" />
              <br><br>
              <button type="submit" style="padding:10px 20px; background:#3498db; color:white; border:none; border-radius:5px; font-size:16px;">
                Verificar
              </button>
            </form>
            <p style="margin-top:20px;"><a href="/login.html">← Voltar</a></p>
          </div>
        \`);
      } catch (emailErr) {
        console.error('Erro ao enviar e-mail:', emailErr);
        res.status(500).send('<script>alert("Erro ao enviar e-mail. Tente novamente."); window.history.back();</script>');
      }
    });
  });
});

// Verificação de código
app.post('/verificar-codigo', csrfProtection, (req, res) => {
  const { usuario_id, codigo } = req.body;
  const sql = \`SELECT * FROM codigos_verificacao WHERE usuario_id = ? AND codigo = ? AND usado = 0 AND datetime(criado_em) >= datetime('now', '-10 minutes')\`;
  db.get(sql, [usuario_id, codigo], (err, row) => {
    if (err || !row) return res.status(400).send('<script>alert("Código inválido, expirado ou já usado."); window.history.back();</script>');
    db.run('UPDATE codigos_verificacao SET usado = 1 WHERE id = ?', [row.id]);
    req.session.userId = usuario_id;
    res.redirect('/');
  });
});

// Logout
app.post('/logout', csrfProtection, (req, res) => {
  req.session.destroy(() => res.redirect('/login.html'));
});

// Recuperação de senha
app.post('/forgot-password', csrfProtection, async (req, res) => {
  const { cpf } = req.body;
  const cpfLimpo = cpf.replace(/[^\d]/g, '');
  if (!validarCPF(cpfLimpo)) return res.status(400).send('<script>alert("CPF inválido!"); window.history.back();</script>');
  db.get('SELECT id FROM usuarios WHERE cpf = ?', [cpfLimpo], async (err, usuario) => {
    if (err || !usuario) return res.status(400).send('<script>alert("CPF não encontrado."); window.history.back();</script>');
    const token = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
    const expiresAt = new Date(Date.now() + 15 * 60 * 1000);
    db.run('INSERT INTO password_resets (usuario_id, token, expires_at) VALUES (?, ?, ?)', [usuario.id, token, expiresAt], function(err) {
      if (err) return res.status(500).send('<script>alert("Erro ao gerar token."); window.history.back();</script>');
      res.send(\`
        <div style="padding: 2rem; text-align: center;">
          <h2>✅ Link de recuperação gerado!</h2>
          <p>Clique no link abaixo para redefinir sua senha:</p>
          <a href="/reset-password.html?token=\${token}" style="color: #3498db; font-weight: bold;">Redefinir Senha</a>
          <br><br>
          <a href="/login.html">Voltar ao login</a>
        </div>
      \`);
    });
  });
});

// Redefinição de senha
app.post('/reset-password', csrfProtection, async (req, res) => {
  const { token, nova_senha, confirmar_senha } = req.body;
  if (nova_senha !== confirmar_senha) return res.status(400).send('<script>alert("Senhas não coincidem."); window.history.back();</script>');
  if (nova_senha.length < 6) return res.status(400).send('<script>alert("Senha deve ter pelo menos 6 caracteres."); window.history.back();</script>');
  db.get('SELECT * FROM password_resets WHERE token = ? AND expires_at > datetime("now")', [token], async (err, row) => {
    if (err || !row) return res.status(400).send('<script>alert("Token inválido ou expirado."); window.location.href="/login.html";</script>');
    try {
      const hashedSenha = await bcrypt.hash(nova_senha, 10);
      db.run('UPDATE usuarios SET senha = ? WHERE id = ?', [hashedSenha, row.usuario_id], function(err) {
        if (err) return res.status(500).send('<script>alert("Erro ao atualizar senha."); window.history.back();</script>');
        db.run('DELETE FROM password_resets WHERE token = ?', [token]);
        res.send(\`
          <div style="padding: 2rem; text-align: center;">
            <h2>✅ Senha redefinida com sucesso!</h2>
            <p><a href="/login.html">Faça login com sua nova senha</a></p>
          </div>
        \`);
      });
    } catch (err) {
      res.status(500).send('<script>alert("Erro interno."); window.history.back();</script>');
    }
  });
});

// API de Bíblia
app.get('/api/bible', requireAuth, async (req, res) => {
  const { reference } = req.query;
  if (!reference) return res.status(400).json({ error: 'Referência necessária' });
  try {
    const response = await fetch(\`https://bible-api.com/\${encodeURIComponent(reference)}?translation=almeida\`);
    const data = await response.json();
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: 'Erro ao buscar versículo' });
  }
});

// Histórico
app.get('/api/historico', requireAuth, (req, res) => {
  const sql = \`SELECT referencia, texto, data_leitura FROM historico_leitura WHERE usuario_id = ? ORDER BY data_leitura DESC LIMIT 20\`;
  db.all(sql, [req.session.userId], (err, rows) => {
    if (err) return res.status(500).json({ error: 'Erro ao carregar histórico' });
    res.json(rows);
  });
});

// Limpar histórico
app.delete('/api/historico', requireAuth, csrfProtection, (req, res) => {
  db.run('DELETE FROM historico_leitura WHERE usuario_id = ?', [req.session.userId], (err) => {
    if (err) return res.status(500).json({ error: 'Erro ao limpar histórico' });
    res.json({ success: true });
  });
});

app.listen(PORT, () => {
  console.log(`🚀 Servidor rodando em http://localhost:${PORT}`);
  console.log(`📧 E-mails enviados para: ${process.env.GMAIL_USER}`);
});