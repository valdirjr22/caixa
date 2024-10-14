<caixa mercadinho>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Estoque - Supermercado</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        #overlay {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
        }
        .modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 20px;
            border-radius: 5px;
            min-width: 300px;
        }
    </style>
</head>
<body>

<h1>Centro de Distribuição</h1>
<div id="loginScreen" class="modal">
    <h2>Login</h2>
    <input type="text" id="username" placeholder="Usuário" required>
    <input type="password" id="password" placeholder="Senha" required>
    <button id="loginBtn">Entrar</button>
    <p id="loginError" style="color: red; display: none;">Usuário ou senha inválidos.</p>
</div>

<div id="mainContent" style="display:none;">
    <h2>Informações do Estabelecimento</h2>
    <form id="establishmentForm">
        <input type="text" id="establishmentName" placeholder="Nome do Estabelecimento" required>
        <input type="text" id="establishmentCNPJ" placeholder="CNPJ" required>
        <input type="text" id="establishmentAddress" placeholder="Endereço" required>
        <input type="email" id="establishmentEmail" placeholder="Email" required>
        <input type="text" id="establishmentPhone" placeholder="Telefone" required>
        <button type="submit">Salvar Informações</button>
    </form>

    <h2>Cadastro de Produtos</h2>
    <form id="productForm">
        <input type="text" id="productName" placeholder="Nome do Produto" required>
        <input type="number" id="productQuantity" placeholder="Quantidade" required>
        <input type="text" id="productBrand" placeholder="Marca" required>
        <input type="text" id="productLocation" placeholder="Localização" required>
        <input type="number" step="0.01" id="productPrice" placeholder="Preço" required>
        <input type="text" id="productBarcode" placeholder="Código de Barras" required>
        <button type="submit">Cadastrar Produto</button>
    </form>

    <h2>Lista de Produtos</h2>
    <table id="productTable">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Quantidade</th>
                <th>Marca</th>
                <th>Localização</th>
                <th>Preço</th>
                <th>Código de Barras</th>
                <th>Ações</th>
            </tr>
        </thead>
        <tbody>
            <!-- Produtos cadastrados aparecerão aqui -->
        </tbody>
    </table>

    <h2>Vendas</h2>
    <button id="startSaleBtn">Iniciar Venda</button>
    <button id="viewSalesHistoryBtn">Histórico de Vendas</button>

    <div id="overlay">
        <div class="modal" id="saleScreen" style="display:none;">
            <h2>Venda de Produto</h2>
            <input type="text" id="saleIdentifier" placeholder="Código de Barras ou Nome do Produto" required>
            <input type="number" id="saleQuantity" placeholder="Quantidade" required>
            <button id="addProductToSaleBtn">Adicionar Produto</button>
            <button id="completeSaleBtn" style="display:none;">Finalizar Venda</button>
            <button id="cancelSaleBtn">Cancelar Venda</button>
            <div id="addedProductsList" style="margin-top: 10px;"></div>
            <div id="totalSaleValue" style="margin-top: 10px;"><strong>Valor Total: R$ <span id="totalValue">0.00</span></strong></div>
        </div>

        <div class="modal" id="receiptScreen" style="display:none;">
            <h2>Comprovante de Venda</h2>
            <div id="receiptProducts"></div>
            <p><strong>Valor Total:</strong> R$ <span id="receiptTotalValue"></span></p>
            <p><strong>Valor Pago:</strong> R$ <span id="receiptPayment"></span></p>
            <p><strong>Troco:</strong> R$ <span id="receiptChange"></span></p>
            <p><strong>Formas de Pagamento:</strong> <span id="receiptPaymentMethods"></span></p>
            <p><strong>Data:</strong> <span id="receiptDate"></span></p>
            <p><strong>Nome do Estabelecimento:</strong> <span id="receiptEstablishmentName"></span></p>
            <p><strong>CNPJ:</strong> <span id="receiptEstablishmentCNPJ"></span></p>
            <p><strong>Endereço:</strong> <span id="receiptEstablishmentAddress"></span></p>
            <p><strong>Email:</strong> <span id="receiptEstablishmentEmail"></span></p>
            <p><strong>Telefone:</strong> <span id="receiptEstablishmentPhone"></span></p>
            <button id="printReceiptBtn">Imprimir Comprovante</button>
            <button id="closeReceiptBtn">Fechar</button>
        </div>

        <div class="modal" id="salesHistoryScreen" style="display:none;">
            <h2>Histórico de Vendas</h2>
            <table id="salesHistoryTable">
                <thead>
                    <tr>
                        <th>Produto</th>
                        <th>Quantidade</th>
                        <th>Valor Pago</th>
                        <th>Troco</th>
                        <th>Data</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Histórico de vendas aparecerá aqui -->
                </tbody>
            </table>
            <button id="closeSalesHistoryBtn">Fechar</button>
        </div>
    </div>
</div>

<script>
    const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
    const products = JSON.parse(localStorage.getItem('products')) || [];
    let totalSale = 0;
    let addedProducts = [];
    let paymentMethods = [];

    // Função para salvar dados no Local Storage
    function saveData() {
        localStorage.setItem('salesHistory', JSON.stringify(salesHistory));
        localStorage.setItem('products', JSON.stringify(products));
    }

    // Atualizar a tabela de produtos
    function updateProductTable() {
        const tbody = document.querySelector('#productTable tbody');
        tbody.innerHTML = '';
        products.forEach((product, index) => {
            tbody.innerHTML += `
                <tr>
                    <td>${product.name}</td>
                    <td>${product.quantity}</td>
                    <td>${product.brand}</td>
                    <td>${product.location}</td>
                    <td>${product.price.toFixed(2)}</td>
                    <td>${product.barcode}</td>
                    <td><button onclick="removeProduct(${index})">Remover</button></td>
                </tr>
            `;
        });
    }

    // Remover produto da lista
    function removeProduct(index) {
        products.splice(index, 1);
        saveData();
        updateProductTable();
    }

    // Login do usuário
    document.getElementById('loginBtn').addEventListener('click', function() {
        const username = document.getElementById('username').value;
        const password = document.getElementById('password').value;

        if (username === "loja01" && password === "010203") {
            document.getElementById('loginScreen').style.display = 'none';
            document.getElementById('mainContent').style.display = 'block';
            updateProductTable(); // Atualiza a tabela ao logar
        } else {
            document.getElementById('loginError').style.display = 'block';
        }
    });

    // Salvar informações do estabelecimento
    document.getElementById('establishmentForm').addEventListener('submit', function(event) {
        event.preventDefault();
        alert('Informações do estabelecimento salvas com sucesso!');
    });

    // Cadastrar produtos
    document.getElementById('productForm').addEventListener('submit', function(event) {
        event.preventDefault();
        const newProduct = {
            name: document.getElementById('productName').value,
            quantity: parseInt(document.getElementById('productQuantity').value),
            brand: document.getElementById('productBrand').value,
            location: document.getElementById('productLocation').value,
            price: parseFloat(document.getElementById('productPrice').value),
            barcode: document.getElementById('productBarcode').value
        };
        products.push(newProduct);
        saveData();
        updateProductTable();
        document.getElementById('productForm').reset();
    });

    // Iniciar venda
    document.getElementById('startSaleBtn').addEventListener('click', function() {
        document.getElementById('saleScreen').style.display = 'block';
        document.getElementById('overlay').style.display = 'block';
        totalSale = 0;
        addedProducts = [];
        paymentMethods = []; // Resetar métodos de pagamento
        document.getElementById('addedProductsList').innerHTML = '';
        document.getElementById('totalValue').innerText = '0.00';
        document.getElementById('completeSaleBtn').style.display = 'none';
    });

    // Adicionar produto à venda
    document.getElementById('addProductToSaleBtn').addEventListener('click', function() {
        const identifier = document.getElementById('saleIdentifier').value;
        const quantity = parseInt(document.getElementById('saleQuantity').value);
        const product = products.find(p => p.name === identifier || p.barcode === identifier);

        if (product && product.quantity >= quantity) {
            product.quantity -= quantity;
            addedProducts.push({ ...product, quantity });
            totalSale += product.price * quantity;

            document.getElementById('addedProductsList').innerHTML += `
                <div>${product.name} (x${quantity}) - R$ ${(product.price * quantity).toFixed(2)}</div>
            `;
            document.getElementById('totalValue').innerText = totalSale.toFixed(2);
            document.getElementById('saleIdentifier').value = '';
            document.getElementById('saleQuantity').value = '';

            if (totalSale > 0) {
                document.getElementById('completeSaleBtn').style.display = 'block';
            }

            saveData(); // Salvar dados após a venda
        } else {
            alert('Produto não encontrado ou quantidade insuficiente.');
        }
    });

    // Finalizar venda
    document.getElementById('completeSaleBtn').addEventListener('click', function() {
        const paymentMethods = [];
        let totalPayment = 0;

        while (true) {
            const method = prompt("Informe a forma de pagamento (Cartão Débito, Cartão Crédito, etc.):");
            if (!method) break;

            const value = parseFloat(prompt("Informe o valor:"));
            if (isNaN(value) || value <= 0) {
                alert("Valor inválido!");
                continue;
            }

            paymentMethods.push({ method, value });
            totalPayment += value;

            const more = prompt("Adicionar outro pagamento? (s/n)");
            if (more.toLowerCase() !== 's') break;
        }

        if (totalPayment < totalSale) {
            alert("O valor pago é inferior ao total da venda!");
            return;
        }

        const change = totalPayment - totalSale;
        const receiptDate = new Date().toLocaleString();
        document.getElementById('receiptProducts').innerHTML = addedProducts.map(p => `${p.name} (x${p.quantity}) - R$ ${(p.price * p.quantity).toFixed(2)}`).join('<br>');
        document.getElementById('receiptTotalValue').innerText = totalSale.toFixed(2);
        document.getElementById('receiptPayment').innerText = totalPayment.toFixed(2);
        document.getElementById('receiptChange').innerText = change.toFixed(2);
        document.getElementById('receiptDate').innerText = receiptDate;
        
        // Mostrar formas de pagamento no recibo
        document.getElementById('receiptPaymentMethods').innerHTML = paymentMethods.map(pm => `${pm.method}: R$ ${pm.value.toFixed(2)}`).join('<br>');

        document.getElementById('receiptEstablishmentName').innerText = document.getElementById('establishmentName').value;
        document.getElementById('receiptEstablishmentCNPJ').innerText = document.getElementById('establishmentCNPJ').value;
        document.getElementById('receiptEstablishmentAddress').innerText = document.getElementById('establishmentAddress').value;
        document.getElementById('receiptEstablishmentEmail').innerText = document.getElementById('establishmentEmail').value;
        document.getElementById('receiptEstablishmentPhone').innerText = document.getElementById('establishmentPhone').value;

        document.getElementById('saleScreen').style.display = 'none';
        document.getElementById('receiptScreen').style.display = 'block';
        document.getElementById('overlay').style.display = 'block';

        salesHistory.push({ products: addedProducts, total: totalSale, payment: totalPayment, change, date: receiptDate, paymentMethods });
        saveData(); // Salvar histórico de vendas
    });

    // Imprimir recibo
    document.getElementById('printReceiptBtn').addEventListener('click', function() {
        window.print();
    });

    // Fechar recibo
    document.getElementById('closeReceiptBtn').addEventListener('click', function() {
        document.getElementById('receiptScreen').style.display = 'none';
        document.getElementById('overlay').style.display = 'none';
    });

    // Exibir histórico de vendas
    document.getElementById('viewSalesHistoryBtn').addEventListener('click', function() {
        document.getElementById('salesHistoryScreen').style.display = 'block';
        document.getElementById('overlay').style.display = 'block';
        updateSalesHistoryTable();
    });

    // Atualizar tabela de histórico de vendas
    function updateSalesHistoryTable() {
        const tbody = document.querySelector('#salesHistoryTable tbody');
        tbody.innerHTML = '';
        salesHistory.forEach(sale => {
            sale.products.forEach(product => {
                tbody.innerHTML += `
                    <tr>
                        <td>${product.name}</td>
                        <td>${product.quantity}</td>
                        <td>R$ ${sale.total.toFixed(2)}</td>
                        <td>R$ ${sale.change.toFixed(2)}</td>
                        <td>${sale.date}</td>
                    </tr>
                `;
            });
        });
    }

    // Fechar histórico de vendas
    document.getElementById('closeSalesHistoryBtn').addEventListener('click', function() {
        document.getElementById('salesHistoryScreen').style.display = 'none';
        document.getElementById('overlay').style.display = 'none';
    });

    // Cancelar venda
    document.getElementById('cancelSaleBtn').addEventListener('click', function() {
        document.getElementById('saleScreen').style.display = 'none';
        document.getElementById('overlay').style.display = 'none';
    });

    // Atualizar tabela de produtos ao carregar a página
    updateProductTable();
</script>
</body>
</html>
