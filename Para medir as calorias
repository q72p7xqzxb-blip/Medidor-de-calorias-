<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contador de Calorias por IA</title>
    <!-- Carrega Tailwind CSS para estilização moderna e responsiva -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Configuração de fonte -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f7f7;
            touch-action: manipulation; /* Melhora a resposta ao toque em iOS */
        }
        /* Estilo para focar o conteúdo no centro do celular */
        .container {
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            padding: 1rem;
        }
        .result-box {
            white-space: pre-wrap; /* Preserva quebras de linha e espaços no resultado */
        }
    </style>
</head>
<body>
    <div class="container mx-auto max-w-lg">
        <header class="text-center py-6">
            <h1 class="text-3xl font-extrabold text-blue-800">Contador de Calorias 🥗</h1>
            <p class="text-gray-600 mt-1">Análise de Prato por IA</p>
        </header>

        <main class="flex-grow">
            <!-- Cartão Principal de Entrada -->
            <div class="bg-white p-6 rounded-xl shadow-lg mb-6">
                <p class="text-sm text-gray-500 mb-4">Descreva o prato ou carregue uma imagem para a estimativa:</p>

                <!-- Área de Entrada de Texto -->
                <textarea id="foodDescription" rows="3"
                          class="w-full p-3 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:ring-blue-500 transition duration-150 ease-in-out resize-none mb-4"
                          placeholder="Ex: 100g de frango grelhado, 1/2 xícara de arroz integral e salada de alface."></textarea>

                <!-- Input de Arquivo (para imagens) -->
                <label for="foodImage" class="block text-sm font-medium text-gray-700 mb-2">Ou carregue uma foto:</label>
                <input type="file" id="foodImage" accept="image/*"
                       class="w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100 mb-4">

                <!-- Botão de Análise -->
                <button onclick="analyzeFood()" id="analyzeButton"
                        class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-xl shadow-md transition duration-200 ease-in-out disabled:opacity-50">
                    Analisar Calorias
                </button>
            </div>

            <!-- Cartão de Resultados -->
            <div id="resultCard" class="hidden bg-green-50 p-6 rounded-xl shadow-lg border-2 border-green-200">
                <h2 class="text-xl font-bold text-green-800 mb-4">Resultado da Análise</h2>
                <div id="loadingIndicator" class="text-center p-4 hidden">
                    <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500 mx-auto"></div>
                    <p class="mt-2 text-blue-600">Calculando a estimativa...</p>
                </div>
                <pre id="calorieResult" class="result-box text-gray-800 text-base"></pre>
            </div>

            <!-- Cartão de Erro -->
            <div id="errorCard" class="hidden bg-red-50 p-4 rounded-xl shadow-md border-2 border-red-200 mt-4">
                <p id="errorMessage" class="text-red-700 font-medium"></p>
            </div>
        </main>

        <footer class="text-center text-xs text-gray-400 py-4 mt-8">
            <p>Alimentado por Gemini. As estimativas de calorias são aproximadas.</p>
        </footer>
    </div>

    <!-- Firebase Imports (MANDATORY for web apps in this environment) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Define variáveis globais no escopo do window
        window.app = null;
        window.db = null;
        window.auth = null;
        window.userId = null;
        
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        setLogLevel('debug'); // Ativa o log de debug para Firestore

        // Função de inicialização
        async function initFirebase() {
            try {
                if (Object.keys(firebaseConfig).length > 0) {
                    window.app = initializeApp(firebaseConfig);
                    window.db = getFirestore(window.app);
                    window.auth = getAuth(window.app);

                    if (initialAuthToken) {
                        await signInWithCustomToken(window.auth, initialAuthToken);
                    } else {
                        await signInAnonymously(window.auth);
                    }
                    window.userId = window.auth.currentUser?.uid || crypto.randomUUID();
                } else {
                    console.error("Firebase config is missing. App will run without persistent storage/auth.");
                }
            } catch (error) {
                console.error("Firebase initialization or sign-in failed:", error);
                // Permite que o app continue rodando, mas sem funcionalidades de banco de dados
            }
        }

        initFirebase();
    </script>

    <!-- Gemini API Logic -->
    <script>
        const API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent";
        const apiKey = ""; // Chave da API (será injetada pelo ambiente)

        const foodDescriptionInput = document.getElementById('foodDescription');
        const foodImageInput = document.getElementById('foodImage');
        const analyzeButton = document.getElementById('analyzeButton');
        const loadingIndicator = document.getElementById('loadingIndicator');
        const resultCard = document.getElementById('resultCard');
        const calorieResult = document.getElementById('calorieResult');
        const errorCard = document.getElementById('errorCard');
        const errorMessage = document.getElementById('errorMessage');

        /**
         * Converte um arquivo File (imagem) para string Base64.
         * @param {File} file O objeto File da imagem.
         * @returns {Promise<string>} A string Base64.
         */
        function fileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = () => resolve(reader.result.split(',')[1]); // Pega apenas a parte Base64
                reader.onerror = error => reject(error);
                reader.readAsDataURL(file);
            });
        }

        /**
         * Realiza a chamada da API Gemini com a descrição e/ou imagem.
         */
        async function analyzeFood() {
            // 1. Resetar UI e Obter Dados
            resultCard.classList.add('hidden');
            errorCard.classList.add('hidden');
            calorieResult.textContent = '';
            analyzeButton.disabled = true;
            loadingIndicator.classList.remove('hidden');

            const description = foodDescriptionInput.value.trim();
            const imageFile = foodImageInput.files[0];

            if (!description && !imageFile) {
                loadingIndicator.classList.add('hidden');
                analyzeButton.disabled = false;
                errorMessage.textContent = 'Por favor, insira uma descrição do prato ou carregue uma imagem.';
                errorCard.classList.remove('hidden');
                return;
            }

            try {
                // 2. Preparar Conteúdo
                const contents = [];
                let base64Image = null;

                if (imageFile) {
                    // Se houver imagem, converte para Base64
                    base64Image = await fileToBase64(imageFile);
                }

                // Texto base para a IA (o prompt principal)
                const userPrompt = description || "Analise esta imagem de prato de comida. Por favor, estime o conteúdo nutricional e calórico.";

                const parts = [
                    { text: userPrompt }
                ];

                if (base64Image) {
                    // Adiciona a imagem Base64 como parte
                    parts.push({
                        inlineData: {
                            mimeType: imageFile.type,
                            data: base64Image
                        }
                    });
                }
                
                contents.push({ role: "user", parts: parts });

                // 3. Configuração do Gemini
                const systemPrompt = `Você é um assistente de nutrição e cálculo de calorias especializado. Sua tarefa é analisar o prato fornecido (via descrição ou imagem) e estimar o valor calórico total (kcal) e uma breve lista de componentes. Seja conciso e use o idioma Português. 
                Responda estritamente com o nome do prato, o total de calorias e a lista de detalhes, seguindo este formato:
                Estimativa Calórica para [Nome do Prato, ex: Bife com Batata Doce]
                Total Estimado: XXX kcal
                
                Detalhes (Estes valores são aproximados):
                - [Componente 1]: YYY kcal
                - [Componente 2]: ZZZ kcal
                - [Componente 3]: WWW kcal`;
                
                const payload = {
                    contents: contents,
                    systemInstruction: {
                        parts: [{ text: systemPrompt }]
                    },
                    // Usar Google Search para melhor precisão nutricional
                    tools: [{ "google_search": {} }],
                };

                // 4. Chamada da API (com Retentativas)
                const headers = { 'Content-Type': 'application/json' };
                let response = null;
                let maxRetries = 3;
                let delay = 1000; // 1 segundo

                for (let i = 0; i < maxRetries; i++) {
                    try {
                        response = await fetch(`${API_URL}?key=${apiKey}`, {
                            method: 'POST',
                            headers: headers,
                            body: JSON.stringify(payload)
                        });

                        if (response.ok) {
                            break; // Sucesso, sai do loop
                        } else if (response.status === 429 && i < maxRetries - 1) {
                            // Erro de Rate Limit, tenta novamente após delay
                            await new Promise(resolve => setTimeout(resolve, delay));
                            delay *= 2; // Exponential backoff
                        } else {
                            throw new Error(`Erro HTTP: ${response.status}`);
                        }
                    } catch (error) {
                        if (i === maxRetries - 1) throw error; // Se falhar na última tentativa, lança o erro
                        await new Promise(resolve => setTimeout(resolve, delay));
                        delay *= 2;
                    }
                }
                
                if (!response || !response.ok) {
                    throw new Error("Falha na comunicação com a API após várias tentativas.");
                }

                const result = await response.json();
                const text = result.candidates?.[0]?.content?.parts?.[0]?.text || "Não foi possível gerar uma estimativa detalhada. Tente refinar a descrição.";

                // 5. Exibir Resultado
                calorieResult.textContent = text;
                resultCard.classList.remove('hidden');
                
                // Limpa o input de arquivo após a análise
                foodImageInput.value = '';

            } catch (error) {
                console.error("Erro na análise de calorias:", error);
                errorMessage.textContent = `Ocorreu um erro ao calcular as calorias: ${error.message}.`;
                errorCard.classList.remove('hidden');
            } finally {
                // 6. Finalizar UI
                loadingIndicator.classList.add('hidden');
                analyzeButton.disabled = false;
            }
        }

        // Expondo a função para o escopo global (necessário para o atributo onclick no HTML)
        window.analyzeFood = analyzeFood;
    </script>
</body>
</html>
