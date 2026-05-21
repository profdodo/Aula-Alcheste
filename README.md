<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Sistema Escolar</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>

/* RESET */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* FUNDO */
body {
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(135deg, #003366, #0059b3);
}

/* TELAS */
.tela { display: none; }
.ativa { display: block; }

/* CARD */
.container {
    background: white;
    width: 380px;
    margin: 40px auto;
    padding: 25px;
    border-radius: 20px;
    box-shadow: 0 15px 40px rgba(0,0,0,0.3);
}

/* TÍTULO */
h2 {
    text-align: center;
    color: #003366;
    margin-bottom: 10px;
}

/* INPUT */
input {
    width: 100%;
    padding: 12px;
    margin: 5px 0;
    border-radius: 10px;
    border: 1px solid #ccc;
}

/* BOTÃO */
button {
    width: 100%;
    padding: 12px;
    margin-top: 5px;
    border: none;
    border-radius: 10px;
    background: linear-gradient(135deg, #003366, #0059b3);
    color: white;
    cursor: pointer;
    transition: 0.3s;
}

button:hover {
    transform: scale(1.05);
}

/* MENU */
.menu {
    display: flex;
    gap: 5px;
}

/* LISTA */
.item {
    background: #f4f6f9;
    padding: 10px;
    border-radius: 10px;
    margin-top: 8px;
    display: flex;
    justify-content: space-between;
}

.aprovado { color: green; }
.reprovado { color: red; }

</style>
</head>

<body>

<!-- LOGIN -->
<div id="login" class="container tela">
    <h2>🔐 Login Professor</h2>
    <input id="user" placeholder="Usuário">
    <input id="pass" type="password" placeholder="Senha">
    <button onclick="entrar()">Entrar</button>
</div>

<!-- SISTEMA -->
<div id="app" class="container tela">

    <h2>🏫 Sistema Escolar</h2>

    <div class="menu">
        <button onclick="mostrar('notas')">Notas</button>
        <button onclick="mostrar('grafico')">Gráfico</button>
        <button onclick="mostrar('oc')">Ocorrências</button>
    </div>

    <!-- NOTAS -->
    <div id="notas" class="tela">
        <input id="nome" placeholder="Nome do aluno">
        <input id="nota" type="number" placeholder="Nota">
        <button onclick="addAluno()">Adicionar</button>
        <div id="lista"></div>
    </div>

    <!-- GRÁFICO -->
    <div id="grafico" class="tela">
        <canvas id="chart"></canvas>
    </div>

    <!-- OCORRÊNCIAS -->
    <div id="oc" class="tela">
        <input id="alunoOc" placeholder="Aluno">
        <input id="textoOc" placeholder="Ocorrência">
        <button onclick="addOc()">Registrar</button>
        <div id="listaOc"></div>
    </div>

    <button onclick="sair()">Sair</button>
</div>

<script>

// CONFIG
const USER = "professor";
const PASS = "1234";

// ELEMENTOS
const loginDiv = document.getElementById("login");
const appDiv = document.getElementById("app");
const lista = document.getElementById("lista");
const listaOc = document.getElementById("listaOc");
const chartCanvas = document.getElementById("chart");

// DADOS
let alunos = JSON.parse(localStorage.getItem("alunos")) || [];
let ocorrencias = JSON.parse(localStorage.getItem("oc")) || [];

// INICIAR
init();

function init() {
    if (localStorage.getItem("login")) {
        abrirApp();
    } else {
        abrirLogin();
    }
}

// LOGIN
function entrar() {
    const usuario = user.value.trim();
    const senha = pass.value.trim();

    if (!usuario || !senha) {
        alert("Preencha os campos!");
        return;
    }

    if (usuario === USER && senha === PASS) {
        localStorage.setItem("login", true);
        abrirApp();
    } else {
        alert("Login incorreto!");
    }
}

function sair() {
    localStorage.removeItem("login");
    abrirLogin();
}

// TELAS
function abrirLogin() {
    loginDiv.classList.add("ativa");
    appDiv.classList.remove("ativa");
}

function abrirApp() {
    loginDiv.classList.remove("ativa");
    appDiv.classList.add("ativa");

    mostrar("notas");
    atualizarAlunos();
    atualizarGrafico();
    atualizarOc();
}

function mostrar(secao) {
    document.querySelectorAll(".tela").forEach(t => t.classList.remove("ativa"));
    document.getElementById(secao).classList.add("ativa");
}

// ALUNOS
function addAluno() {
    const nomeAluno = nome.value.trim();
    const notaAluno = Number(nota.value);

    if (!nomeAluno || isNaN(notaAluno)) {
        alert("Dados inválidos!");
        return;
    }

    alunos.push({ nome: nomeAluno, nota: notaAluno });

    salvar();
    atualizarAlunos();
    atualizarGrafico();

    nome.value = "";
    nota.value = "";
}

function atualizarAlunos() {
    lista.innerHTML = "";

    alunos.forEach(a => {
        const status = a.nota >= 6 ? "aprovado" : "reprovado";

        lista.innerHTML += `
            <div class="item">
                <span>${a.nome} - ${a.nota}</span>
                <span class="${status}">
                    ${status === "aprovado" ? "✔" : "✖"}
                </span>
            </div>
        `;
    });
}

// GRÁFICO
let chart;

function atualizarGrafico() {
    const nomes = alunos.map(a => a.nome);
    const notas = alunos.map(a => a.nota);

    if (chart) chart.destroy();

    chart = new Chart(chartCanvas, {
        type: "bar",
        data: {
            labels: nomes,
            datasets: [{
                label: "Notas",
                data: notas
            }]
        }
    });
}

// OCORRÊNCIAS
function addOc() {
    const aluno = alunoOc.value.trim();
    const texto = textoOc.value.trim();

    if (!aluno || !texto) {
        alert("Preencha tudo!");
        return;
    }

    ocorrencias.push({
        aluno,
        texto,
        data: new Date().toLocaleDateString()
    });

    salvar();
    atualizarOc();

    alunoOc.value = "";
    textoOc.value = "";
}

function atualizarOc() {
    listaOc.innerHTML = "";

    ocorrencias.forEach(o => {
        listaOc.innerHTML += `
            <div class="item">
                ⚠️ ${o.aluno}: ${o.texto} (${o.data})
            </div>
        `;
    });
}

// STORAGE
function salvar() {
    localStorage.setItem("alunos", JSON.stringify(alunos));
    localStorage.setItem("oc", JSON.stringify(ocorrencias));
}

</script>

</body>
</html>