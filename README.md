<!DOCTYPE html>
<html lang="pt">
<head>
  <meta charset="UTF-8">
  <title>Gestão de Farmácia</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #eaf6f6;
      margin: 0;
      padding: 0;
    }
    header {
      background: #2e8b57;
      color: white;
      padding: 20px;
      text-align: center;
    }
    .container {
      max-width: 600px;
      margin: 20px auto;
      background: white;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    h2 {
      color: #2e8b57;
    }
    input, select, button {
      width: 100%;
      padding: 10px;
      margin-top: 8px;
      margin-bottom: 15px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      background: #2e8b57;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background: #246b45;
    }
    .hidden { display: none; }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    th {
      background-color: #2e8b57;
      color: white;
    }
    #mensagemVenda { color: green; font-weight: bold; }
  </style>
</head>
<body>

<header>
  <h1>Gestão de Farmácia</h1>
</header>

<div class="container" id="loginContainer">
  <h2>Login</h2>
  <input type="text" id="usuario" placeholder="Usuário">
  <input type="password" id="senha" placeholder="Senha">
  <button onclick="login()">Entrar</button>
  <div id="erroLogin" style="color: red;"></div>
</div>

<div class="hidden" id="sistemaFarmacia">

  <div class="container">
    <h2>Cadastrar Medicamento</h2>
    <input type="text" id="nomeMedicamento" placeholder="Nome do medicamento">
    <input type="number" id="precoMedicamento" placeholder="Preço (Kz)" min="0">
    <input type="number" id="estoqueMedicamento" placeholder="Estoque inicial" min="0">
    <input type="date" id="validadeMedicamento">
    <button onclick="cadastrarMedicamento()">Cadastrar</button>
  </div>

  <div class="container">
    <h2>Cadastrar Cliente</h2>
    <input type="text" id="nomeCliente" placeholder="Nome do cliente">
    <input type="number" id="saldoCliente" placeholder="Saldo inicial (Kz)" min="0">
    <button onclick="cadastrarCliente()">Cadastrar</button>
  </div>

  <div class="container">
    <h2>Venda de Medicamento</h2>
    <select id="clienteVenda"></select>
    <select id="medicamentoVenda"></select>
    <input type="number" id="quantidadeVenda" placeholder="Quantidade" min="1" value="1">
    <button onclick="realizarVenda()">Vender</button>
    <div id="mensagemVenda"></div>
  </div>

  <div class="container">
    <h2>Relatório de Vendas</h2>
    <button onclick="exportarParaExcel()">Exportar para Excel</button>
    <table id="tabelaExcel">
      <thead>
        <tr>
          <th>Cliente</th>
          <th>Medicamento</th>
          <th>Quantidade</th>
          <th>Total (Kz)</th>
        </tr>
      </thead>
      <tbody id="tabelaVendas"></tbody>
    </table>
  </div>
</div>

<script>
  const usuarioPadrao = "farmaceutico";
  const senhaPadrao = "1234";

  let medicamentos = JSON.parse(localStorage.getItem("medicamentos")) || [];
  let clientes = JSON.parse(localStorage.getItem("clientesFarmacia")) || [];
  let vendas = JSON.parse(localStorage.getItem("vendasFarmacia")) || [];

  function salvarDados() {
    localStorage.setItem("medicamentos", JSON.stringify(medicamentos));
    localStorage.setItem("clientesFarmacia", JSON.stringify(clientes));
    localStorage.setItem("vendasFarmacia", JSON.stringify(vendas));
  }

  function login() {
    const u = document.getElementById("usuario").value;
    const s = document.getElementById("senha").value;
    if (u === usuarioPadrao && s === senhaPadrao) {
      document.getElementById("loginContainer").style.display = "none";
      document.getElementById("sistemaFarmacia").classList.remove("hidden");
      atualizarListas();
    } else {
      document.getElementById("erroLogin").textContent = "Usuário ou senha incorretos!";
    }
  }

  function cadastrarMedicamento() {
    const nome = document.getElementById("nomeMedicamento").value;
    const preco = parseFloat(document.getElementById("precoMedicamento").value);
    const estoque = parseInt(document.getElementById("estoqueMedicamento").value);
    const validade = document.getElementById("validadeMedicamento").value;

    if (!nome || isNaN(preco) || isNaN(estoque) || !validade) return alert("Preencha todos os campos corretamente.");

    medicamentos.push({ nome, preco, estoque, validade });
    salvarDados();
    document.getElementById("nomeMedicamento").value = "";
    document.getElementById("precoMedicamento").value = "";
    document.getElementById("estoqueMedicamento").value = "";
    document.getElementById("validadeMedicamento").value = "";
    atualizarListas();
  }

  function cadastrarCliente() {
    const nome = document.getElementById("nomeCliente").value;
    const saldo = parseFloat(document.getElementById("saldoCliente").value);

    if (!nome || isNaN(saldo)) return alert("Preencha os dados corretamente.");

    clientes.push({ nome, saldo });
    salvarDados();
    document.getElementById("nomeCliente").value = "";
    document.getElementById("saldoCliente").value = "";
    atualizarListas();
  }

  function atualizarListas() {
    let clienteSelect = document.getElementById("clienteVenda");
    let medicamentoSelect = document.getElementById("medicamentoVenda");
    clienteSelect.innerHTML = "";
    medicamentoSelect.innerHTML = "";

    clientes.forEach((c, i) => {
      let op = document.createElement("option");
      op.value = i;
      op.textContent = `${c.nome} - Kz ${c.saldo.toFixed(2)}`;
      clienteSelect.appendChild(op);
    });

    medicamentos.forEach((m, i) => {
      let op = document.createElement("option");
      op.value = i;
      op.textContent = `${m.nome} - Kz ${m.preco.toFixed(2)} (Estoque: ${m.estoque})`;
      medicamentoSelect.appendChild(op);
    });

    atualizarTabelaVendas();
  }

  function realizarVenda() {
    const iCliente = document.getElementById("clienteVenda").value;
    const iMed = document.getElementById("medicamentoVenda").value;
    const qtd = parseInt(document.getElementById("quantidadeVenda").value);

    if (iCliente === "" || iMed === "" || isNaN(qtd) || qtd <= 0) return alert("Dados inválidos.");

    const cliente = clientes[iCliente];
    const medicamento = medicamentos[iMed];
    const total = medicamento.preco * qtd;

    if (medicamento.estoque < qtd) return alert("Estoque insuficiente.");
    if (cliente.saldo < total) return alert("Saldo insuficiente.");

    medicamento.estoque -= qtd;
    cliente.saldo -= total;
    vendas.push({ cliente: cliente.nome, medicamento: medicamento.nome, quantidade: qtd, total });
    salvarDados();

    document.getElementById("mensagemVenda").textContent = `Venda realizada: ${cliente.nome} comprou ${qtd}x ${medicamento.nome} - Kz ${total.toFixed(2)}`;
    atualizarListas();
  }

  function atualizarTabelaVendas() {
    let tabela = document.getElementById("tabelaVendas");
    tabela.innerHTML = "";
    vendas.forEach(v => {
      let tr = document.createElement("tr");
      tr.innerHTML = `<td>${v.cliente}</td><td>${v.medicamento}</td><td>${v.quantidade}</td><td>Kz ${v.total.toFixed(2)}</td>`;
      tabela.appendChild(tr);
    });
  }

  function exportarParaExcel() {
    let tabela = document.getElementById("tabelaExcel").outerHTML;
    let blob = new Blob(["\ufeff", tabela], { type: "application/vnd.ms-excel" });
    let url = URL.createObjectURL(blob);
    let link = document.createElement("a");
    link.href = url;
    link.download = "relatorio_farmacia.xls";
    link.click();
  }
</script>

</body>
</html>
