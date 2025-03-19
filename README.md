
# Gestion-finance

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestion des Finances</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: auto;
            padding: 20px;
            background-color: #f4f4f4;
            display: none; /* Caché tant que l'accès n'est pas validé */
        }
        h2, h3 {
            text-align: center;
        }
        form, #auth-section {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        input, button {
            padding: 10px;
            font-size: 16px;
        }
        #transactions, #monthly-summary {
            margin-top: 20px;
            background: white;
            padding: 10px;
            border-radius: 5px;
        }
        canvas {
            margin-top: 20px;
        }
        #access-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: white;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
        }
    </style>
</head>
<body>
    <!-- Écran d'accès au site -->
    <div id="access-screen">
        <h2>Accès au Site</h2>
        <input type="password" id="access-code" placeholder="Entrez votre code" required>
        <button id="set-access-code">Définir Code</button>
        <button id="enter-site">Entrer</button>
        <p id="access-message"></p>
    </div>

    <!-- Section de connexion aux transactions -->
    <div id="auth-section">
        <h3>Création / Connexion</h3>
        <input type="password" id="user-code" placeholder="Entrez votre code transactionnel" required>
        <button id="set-code">Définir Code</button>
        <button id="login">Se Connecter</button>
        <p id="auth-message"></p>
    </div>

    <!-- Gestion des finances -->
    <h2>Suivi des Finances</h2>
    <form id="transaction-form">
        <input type="number" id="amount" placeholder="Montant" required>
        <input type="date" id="date" required>
        <button type="button" id="add-income">Ajouter Revenu</button>
        <button type="button" id="add-expense">Ajouter Dépense</button>
    </form>
    <div id="transactions">
        <h3>Liste des Transactions</h3>
        <ul id="transaction-list"></ul>
        <h3>Total: <span id="total">0</span>€</h3>
    </div>
    <div id="monthly-summary">
        <h3>Compte-rendu Mensuel</h3>
        <ul id="monthly-summary-list"></ul>
    </div>
    <canvas id="financeChart"></canvas>

    <script>
        // Gestion de l'accès au site
        const accessScreen = document.getElementById('access-screen');
        const accessCodeInput = document.getElementById('access-code');
        const setAccessCodeButton = document.getElementById('set-access-code');
        const enterSiteButton = document.getElementById('enter-site');
        const accessMessage = document.getElementById('access-message');

        function checkAccessCode() {
            if (localStorage.getItem('accessCode')) {
                accessMessage.textContent = "Veuillez entrer votre code pour accéder au site.";
                setAccessCodeButton.style.display = "none";
            } else {
                accessMessage.textContent = "Définissez un code pour sécuriser l'accès.";
            }
        }

        setAccessCodeButton.addEventListener('click', () => {
            const code = accessCodeInput.value.trim();
            if (code.length < 4) {
                alert("Le code doit contenir au moins 4 caractères.");
                return;
            }
            localStorage.setItem('accessCode', code);
            alert("Code enregistré !");
            checkAccessCode();
            accessCodeInput.value = "";
        });

        enterSiteButton.addEventListener('click', () => {
            if (accessCodeInput.value.trim() === localStorage.getItem('accessCode')) {
                accessScreen.style.display = "none";
                document.body.style.display = "block";
            } else {
                alert("Code incorrect !");
            }
            accessCodeInput.value = "";
        });

        checkAccessCode();

        // Gestion de la connexion aux transactions
        const authSection = document.getElementById('auth-section');
        const userCodeInput = document.getElementById('user-code');
        const setCodeButton = document.getElementById('set-code');
        const loginButton = document.getElementById('login');
        const authMessage = document.getElementById('auth-message');
        let isAuthenticated = false;

        function checkUserCode() {
            if (localStorage.getItem('userCode')) {
                authMessage.textContent = "Un code existe déjà. Veuillez vous connecter.";
            } else {
                authMessage.textContent = "Définissez un code transactionnel.";
            }
        }

        setCodeButton.addEventListener('click', () => {
            if (userCodeInput.value.trim().length < 4) {
                alert("Le code doit contenir au moins 4 caractères.");
                return;
            }
            localStorage.setItem('userCode', userCodeInput.value.trim());
            alert("Code transactionnel enregistré !");
            checkUserCode();
            userCodeInput.value = "";
        });

        loginButton.addEventListener('click', () => {
            if (userCodeInput.value.trim() === localStorage.getItem('userCode')) {
                isAuthenticated = true;
                authSection.style.display = "none";
                loadTransactions();
            } else {
                alert("Code incorrect !");
            }
            userCodeInput.value = "";
        });

        checkUserCode();

        // Gestion des transactions
        const form = document.getElementById('transaction-form');
        const transactionList = document.getElementById('transaction-list');
        const totalDisplay = document.getElementById('total');
        let transactions = [];

        function loadTransactions() {
            if (!isAuthenticated) return;
            transactions = JSON.parse(localStorage.getItem('transactions')) || [];
            totalDisplay.textContent = transactions.reduce((acc, t) => acc + (t.type === 'income' ? t.amount : -t.amount), 0);
        }

        function saveTransactions() {
            localStorage.setItem('transactions', JSON.stringify(transactions));
        }

        function addTransaction(type) {
            if (!isAuthenticated) {
                alert("Veuillez vous connecter !");
                return;
            }

            const amount = parseFloat(document.getElementById('amount').value);
            const date = document.getElementById('date').value;

            if (isNaN(amount) || amount <= 0 || !date) {
                alert("Veuillez entrer des informations valides.");
                return;
            }

            transactions.push({ type, amount, date });
            saveTransactions();
            totalDisplay.textContent = transactions.reduce((acc, t) => acc + (t.type === 'income' ? t.amount : -t.amount), 0);
        }

        document.getElementById('add-income').addEventListener('click', () => addTransaction('income'));
        document.getElementById('add-expense').addEventListener('click', () => addTransaction('expense'));

        loadTransactions();
    </script>
</body>
</html>

