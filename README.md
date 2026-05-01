<!DOCTYPE html>
<html lang="pt">
<head>
<meta charset="UTF-8">
<title>Faturamento Simples</title>
<style>
    body { font-family: Arial; background: #f5f5f5; padding: 20px; }
    .box { max-width: 800px; margin: auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #ccc; }
    h1 { text-align: center; color: #333; }
    input, button { padding: 10px; margin: 5px 0; width: 100%; border: 1px solid #ddd; border-radius: 4px; }
    .linha { display: grid; grid-template-columns: 2fr 1fr 1fr 1fr 50px; gap: 10px; align-items: end; }
    button { background: #0066cc; color: white; border: none; cursor: pointer; }
    button:hover { background: #0052a3; }
    table { width: 100%; border-collapse: collapse; margin-top: 20px; }
    th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
    th { background: #333; color: white; }
    .totais { text-align: right; margin-top: 20px; font-size: 18px; }
    .totais p { margin: 5px 0; }
    .total-geral { font-size: 22px; font-weight: bold; color: green; }
    .btn-apagar { background: red; padding: 5px; }
    @media print { .no-print { display: none; } }
</style>
</head>
<body>
<div class="box">
    <h1>Fatura Nº <span id="numFatura">001</span></h1>
    
    <div class="no-print">
        <input type="text" id="cliente" placeholder="Nome do Cliente" oninput="atualizaCliente()">
        <div class="linha">
            <input type="text" id="produto" placeholder="Produto/Serviço">
            <input type="number" id="qtd" placeholder="Qtd" value="1" min="1">
            <input type="number" id="preco" placeholder="Preço Kz" min="0" step="0.01">
            <select id="iva">
                <option value="14">IVA 14%</option>
                <option value="0">Isento</option>
            </select>
            <button onclick="addItem()">+</button>
        </div>
    </div>

    <p><strong>Cliente:</strong> <span id="nomeCliente">---</span></p>
    <p><strong>Data:</strong> <span id="data"></span></p>

    <table>
        <thead>
            <tr>
                <th>Descrição</th>
                <th>Qtd</th>
                <th>Preço</th>
                <th>IVA</th>
                <th>Total</th>
                <th class="no-print"></th>
            </tr>
        </thead>
        <tbody id="corpoTabela"></tbody>
    </table>

    <div class="totais">
        <p>Subtotal: <span id="subtotal">0.00 Kz</span></p>
        <p>IVA: <span id="valorIva">0.00 Kz</span></p>
        <p class="total-geral">TOTAL: <span id="total">0.00 Kz</span></p>
    </div>

    <button class="no-print" onclick="window.print()" style="margin-top:20px; width: 100%;">Imprimir Fatura</button>
</div>

<script>
    let fatura = [];
    document.getElementById('data').textContent = new Date().toLocaleDateString('pt-AO');

    function atualizaCliente() {
        document.getElementById('nomeCliente').textContent = document.getElementById('cliente').value || '---';
    }

    function addItem() {
        const produto = document.getElementById('produto').value;
        const qtd = parseFloat(document.getElementById('qtd').value);
        const preco = parseFloat(document.getElementById('preco').value);
        const taxaIva = parseFloat(document.getElementById('iva').value);

        if (!produto || qtd <= 0 || preco < 0) {
            alert('Preenche produto, qtd e preço');
            return;
        }

        const sub = qtd * preco;
        const ivaValor = sub * (taxaIva / 100);
        const totalItem = sub + ivaValor;

        fatura.push({
            id: Date.now(),
            produto, qtd, preco, 
            taxaIva, ivaValor, 
            sub, totalItem
        });

        desenhaTabela();
        limpaCampos();
    }

    function desenhaTabela() {
        const tbody = document.getElementById('corpoTabela');
        tbody.innerHTML = '';
        
        fatura.forEach(item => {
            tbody.innerHTML += `
                <tr>
                    <td>${item.produto}</td>
                    <td>${item.qtd}</td>
                    <td>${item.preco.toFixed(2)} Kz</td>
                    <td>${item.ivaValor.toFixed(2)} Kz (${item.taxaIva}%)</td>
                    <td><strong>${item.totalItem.toFixed(2)} Kz</strong></td>
                    <td class="no-print"><button class="btn-apagar" onclick="apagaItem(${item.id})">X</button></td>
                </tr>
            `;
        });
        
        calculaTotal();
    }

    function apagaItem(id) {
        fatura = fatura.filter(item => item.id !== id);
        desenhaTabela();
    }

    function calculaTotal() {
        const sub = fatura.reduce((soma, i) => soma + i.sub, 0);
        const iva = fatura.reduce((soma, i) => soma + i.ivaValor, 0);
        const total = sub + iva;

        document.getElementById('subtotal').textContent = sub.toFixed(2) + ' Kz';
        document.getElementById('valorIva').textContent = iva.toFixed(2) + ' Kz';
        document.getElementById('total').textContent = total.toFixed(2) + ' Kz';
    }

    function limpaCampos() {
        document.getElementById('produto').value = '';
        document.getElementById('qtd').value = 1;
        document.getElementById('preco').value = '';
        document.getElementById('produto').focus();
    }
</script>
</body>
</html>
