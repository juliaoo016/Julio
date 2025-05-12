<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Ponto - Administração</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 300px;
            text-align: center;
        }
        h1 {
            font-size: 24px;
            margin-bottom: 20px;
        }
        .input-field {
            margin: 10px 0;
        }
        .input-field input, .input-field select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }
        .button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin-top: 10px;
            width: 100%;
        }
        .button:hover {
            background-color: #45a049;
        }
        .log-table {
            width: 100%;
            margin-top: 20px;
            border-collapse: collapse;
        }
        .log-table th, .log-table td {
            padding: 8px;
            border: 1px solid #ddd;
            text-align: center;
        }
        .log-table th {
            background-color: #4CAF50;
            color: white;
        }
        #registroPonto {
            display: none;
        }
        #loginSection {
            display: block;
        }
    </style>
</head>
<body>

<div class="container" id="loginSection">
    <h1>Controle de Ponto</h1>

    <div class="input-field">
        <input type="text" id="username" placeholder="Nome de usuário" required>
    </div>

    <div class="input-field">
        <input type="password" id="password" placeholder="Senha" required>
    </div>

    <div class="input-field">
        <select id="cargo">
            <option value="">Selecione o Cargo</option>
            <option value="Moderador Iniciante">Moderador Iniciante</option>
            <option value="Moderador Líder">Moderador Líder</option>
            <option value="Moderador Supervisor">Moderador Supervisor</option>
            <option value="Administrador Líder">Administrador Líder</option>
            <option value="Administrador Sub Gerente">Administrador Sub Gerente</option>
            <option value="Administrador Gerente">Administrador Gerente</option>
            <option value="Fundador">Fundador</option>
        </select>
    </div>

    <button class="button" onclick="login()">Login</button>
</div>

<div class="container" id="registroPonto">
    <h1>Controle de Ponto</h1>

    <div class="input-field">
        <input type="text" id="nome" placeholder="Nome do Jogador" disabled>
    </div>

    <div class="input-field">
        <input type="text" id="cargoUsuario" placeholder="Cargo" disabled>
    </div>

    <button class="button" onclick="registrarEntrada()">Iniciar Turno</button>
    <button class="button" onclick="registrarSaida()">Finalizar Turno</button>

    <h2>Histórico de Ponto</h2>
    <table class="log-table" id="tabelaPonto">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Cargo</th>
                <th>Entrada</th>
                <th>Saída</th>
                <th>Horas Trabalhadas</th>
            </tr>
        </thead>
        <tbody>
        </tbody>
    </table>

    <!-- Botão para exportar para CSV -->
    <button class="button" onclick="exportarParaCSV()">Exportar para CSV</button>
</div>

<script>
    let registros = [];
    let usuarioLogado = null;

    // Dados de login (nome de usuário, senha, cargo)
    const usuarios = [
        { nome: "julio", senha: "123", cargo: "Moderador Supervisor" }
    ];

    // Função para realizar o login
    function login() {
        let nome = document.getElementById('username').value;
        let senha = document.getElementById('password').value;
        let cargo = document.getElementById('cargo').value;

        let usuario = usuarios.find(user => user.nome === nome && user.senha === senha && user.cargo === cargo);

        if (usuario) {
            usuarioLogado = usuario;
            document.getElementById('loginSection').style.display = 'none';
            document.getElementById('registroPonto').style.display = 'block';
            document.getElementById('nome').value = usuario.nome;
            document.getElementById('cargoUsuario').value = usuario.cargo;
            alert("Login realizado com sucesso!");
        } else {
            alert("Usuário ou senha incorretos, ou o cargo não corresponde ao seu cargo permitido.");
        }
    }

    // Função para registrar a entrada
    function registrarEntrada() {
        if (!usuarioLogado) {
            alert("Você não está logado!");
            return;
        }

        let dataEntrada = new Date();
        
        let registro = {
            nome: usuarioLogado.nome,
            cargo: usuarioLogado.cargo,
            entrada: dataEntrada.toLocaleString(),
            saida: "-",
            horasTrabalhadas: "-"
        };
        registros.push(registro);
        atualizarTabela();
        alert("Turno iniciado!");
    }

    // Função para registrar a saída
    function registrarSaida() {
        if (!usuarioLogado) {
            alert("Você não está logado!");
            return;
        }

        for (let i = registros.length - 1; i >= 0; i--) {
            if (registros[i].nome === usuarioLogado.nome && registros[i].saida === "-") {
                let dataSaida = new Date();
                let horasTrabalhadas = calcularHoras(registros[i].entrada, dataSaida.toLocaleString());
                registros[i].saida = dataSaida.toLocaleString();
                registros[i].horasTrabalhadas = horasTrabalhadas;
                atualizarTabela();
                alert("Turno finalizado!");
                break;
            }
        }
    }

    // Função para calcular as horas trabalhadas
    function calcularHoras(entrada, saida) {
        let dataEntrada = new Date(entrada);
        let dataSaida = new Date(saida);
        let diff = dataSaida - dataEntrada; // diferença em milissegundos
        let horas = Math.floor(diff / 1000 / 60 / 60);
        let minutos = Math.floor((diff / 1000 / 60) % 60);
        return horas + "h " + minutos + "m";
    }

    // Função para atualizar a tabela de ponto
    function atualizarTabela() {
        let tabela = document.getElementById("tabelaPonto").getElementsByTagName("tbody")[0];
        tabela.innerHTML = "";
        
        registros.forEach(registro => {
            let row = tabela.insertRow();
            row.insertCell(0).innerText = registro.nome;
            row.insertCell(1).innerText = registro.cargo;
            row.insertCell(2).innerText = registro.entrada;
            row.insertCell(3).innerText = registro.saida;
            row.insertCell(4).innerText = registro.horasTrabalhadas;
        });
    }

    // Função para exportar os registros para CSV
    function exportarParaCSV() {
        let csvContent = "Nome,Cargo,Entrada,Saída,Horas Trabalhadas\n";

        registros.forEach(function(registro) {
            let row = `${registro.nome},${registro.cargo},${registro.entrada},${registro.saida},${registro.horasTrabalhadas}\n`;
            csvContent += row;
        });

        // Criar o link para download do arquivo CSV
        let link = document.createElement('a');
        link.setAttribute('href', 'data:text/csv;charset=utf-8,' + encodeURIComponent(csvContent));
        link.setAttribute('download', 'registros_de_ponto.csv');
        link.click();
    }
</script>

</body>
</html>