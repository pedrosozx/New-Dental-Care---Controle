<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Dashboard New Dental Care</title>
  <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; padding: 1rem; background: #f4f4f4; }
    h1, h2 { text-align: center; }
    .resumo { text-align: center; margin-bottom: 1rem; }
    .grid { display: flex; flex-wrap: wrap; gap: 1rem; justify-content: center; }
    .card {
      background: white; padding: 1rem; border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      width: 300px;
    }
    .progress-bar {
      height: 20px; border-radius: 10px; background: #ddd;
      overflow: hidden; margin-top: 0.5rem;
    }
    .progress {
      height: 100%; border-radius: 10px; text-align: center; color: white; font-size: 12px;
    }
  </style>
</head>
<body>
  <h1>Dashboard de Vendas - New Dental Care</h1>
  <div class="resumo" id="resumoGeral"></div>
  <div class="grid" id="dashboard"></div>

  <script>
    const metas = {
      Master: 490000,
      Senior: 280000,
      Pleno: 140000,
      Junior: 70000
    };

    const representantes = {
      Renato: 'Master',
      Djanaina: 'Senior', Anderson: 'Senior', Gilberto: 'Senior',
      Fernanda: 'Pleno', Larissa: 'Pleno', William: 'Pleno', Ygor: 'Pleno',
      Thiago: 'Pleno', Tatiane: 'Pleno', "Caroline e Paulo": 'Pleno'
    };

    const planilhas = {
      clientes: "https://docs.google.com/spreadsheets/d/e/2PACX-1vTOUNJ_AS_BxDaZNTL97c9w0_kLV3IfCckLN3RVt_hiU2VhTr8omsFOL1YCCbOcrL-VBjO8T03GehmM/pub?output=csv",
      nox: "https://docs.google.com/spreadsheets/d/e/2PACX-1vRTwTDVBnT49kn90vlYVUbiUTJEdHx_KsBrzppZnxm3Xi4wfsOXjIb6gcU7PFl9bwBTzj3TiZnmWIt5/pub?output=csv",
      bling: "https://docs.google.com/spreadsheets/d/e/2PACX-1vQZGsxBj0w1Ze9YWV7NUIxM8oe2mGglGrQQ1N_NDwDTCJwJe5uK9XjcEIslDRHQJz1kEJ20WYuw0VIw/pub?output=csv"
    };

    const dados = {
      mapaUsuarios: {}, // cpf/cnpj -> representante
      vendas: {} // representante -> valor total
    };

    function adicionarVenda(rep, valor) {
      if (!rep || isNaN(valor)) return;
      if (!dados.vendas[rep]) dados.vendas[rep] = 0;
      dados.vendas[rep] += valor;
    }

    function carregarDashboard() {
      const dashboard = document.getElementById("dashboard");
      const resumoGeral = document.getElementById("resumoGeral");
      dashboard.innerHTML = "";
      resumoGeral.innerHTML = "";

      const repsOrdenados = Object.entries(dados.vendas).sort((a, b) => {
        const metaA = metas[representantes[a[0]] || 'Junior'];
        const metaB = metas[representantes[b[0]] || 'Junior'];
        return (b[1]/metaB) - (a[1]/metaA);
      });

      const totalGeral = Object.values(dados.vendas).reduce((acc, val) => acc + val, 0);
      resumoGeral.innerHTML = `<h2>Total Geral Vendido: R$ ${totalGeral.toLocaleString(undefined, {minimumFractionDigits: 2})}</h2>`;

      for (const [rep, valor] of repsOrdenados) {
        const nivel = representantes[rep] || 'Junior';
        const meta = metas[nivel];
        const pct = Math.min(100, (valor / meta) * 100);
        const cor = pct >= 90 ? 'green' : pct >= 50 ? 'orange' : 'red';

        const card = document.createElement("div");
        card.className = "card";
        card.innerHTML = `
          <h3>${rep}</h3>
          <p><strong>Nível:</strong> ${nivel}</p>
          <p><strong>Meta:</strong> R$ ${meta.toLocaleString()}</p>
          <p><strong>Vendido:</strong> R$ ${valor.toLocaleString(undefined, {minimumFractionDigits: 2})}</p>
          <div class="progress-bar">
            <div class="progress" style="width:${pct}%;background:${cor}">${pct.toFixed(1)}%</div>
          </div>
        `;
        dashboard.appendChild(card);
      }
    }

    function carregarTudo() {
      dados.vendas = {};
      dados.mapaUsuarios = {};

      Papa.parse(planilhas.clientes, {
        download: true,
        header: true,
        complete: res => {
          res.data.forEach(linha => {
            const cpf = linha["CNPJ/CPF"]?.trim();
            const rep = linha["Representante"]?.trim();
            if (cpf && rep) dados.mapaUsuarios[cpf] = rep;
          });
          carregarNox();
        }
      });
    }

    function carregarNox() {
      Papa.parse(planilhas.nox, {
        download: true,
        header: true,
        complete: res => {
          res.data.forEach(linha => {
            const usuario = linha["Usuário Venda"]?.trim();
            const valor = parseFloat((linha["Valor Total Original"] || "0").replace(/\./g, '').replace(',', '.'));
            const rep = dados.mapaUsuarios[usuario];
            adicionarVenda(rep, valor);
          });
          carregarBling();
        }
      });
    }

    function carregarBling() {
      Papa.parse(planilhas.bling, {
        download: true,
        header: true,
        complete: res => {
          res.data.forEach(linha => {
            const cliente = linha["Nome Cliente"]?.trim();
            const valor = parseFloat((linha["Valor Bruto"] || "0").replace(/\./g, '').replace(',', '.'));
            const rep = dados.mapaUsuarios[cliente];
            adicionarVenda(rep, valor);
          });
          carregarDashboard();
        }
      });
    }

    carregarTudo();
    setInterval(carregarTudo, 10 * 60 * 1000); // Atualiza a cada 10 minutos
  </script>
</body>
</html>
