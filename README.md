index.html<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Osos Escandalosos Chat</title>
    
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts: Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        /* Definición de temas de color */
        :root {
            --color-pardo: #d97706;    /* Naranja/Marrón oscuro */
            --color-panda: #ec4899;    /* Rosa/Magenta */
            --color-polar: #06b6d4;    /* Cian/Azul Claro */
            --bg-chat: #1f1f1f;
            --bg-main: #1a1a1a;
            --text-color: #ffffff;
        }

        /* Estilos base para el modo oscuro */
        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--bg-main);
            color: var(--text-color);
            transition: all 0.3s ease;
        }
        .chat-container {
            background-color: var(--bg-chat);
            max-width: 640px;
            height: 100vh;
            display: flex;
            flex-direction: column;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
        }
        .message-list {
            flex-grow: 1;
            overflow-y: auto;
            padding: 1rem;
            scroll-behavior: smooth;
        }
        .user-message {
            background-color: #444444;
            color: var(--text-color);
            padding: 0.5rem 1rem;
            border-radius: 1.5rem 1.5rem 0.5rem 1.5rem;
            max-width: 85%;
            margin-left: auto;
        }
        .ai-message {
            color: var(--text-color);
            padding: 0.5rem 1rem;
            max-width: 90%;
            margin-right: auto;
            white-space: pre-wrap;
        }
        .input-field {
            resize: none;
            background-color: #333333;
            color: var(--text-color);
            border: none;
            border-radius: 1.5rem;
            padding: 0.75rem 1.5rem;
        }
        .send-button {
            transition: background-color 0.2s;
        }

        /* Clase dinámica para el encabezado y botones */
        .header-color-pardo { background-color: var(--color-pardo); }
        .header-color-panda { background-color: var(--color-panda); }
        .header-color-polar { background-color: var(--color-polar); }

        /* Estilos para el menú de selección de personajes */
        .character-selector {
            position: absolute;
            top: 100%;
            left: 0;
            background-color: #333333;
            border-radius: 0.5rem;
            z-index: 50;
            opacity: 0;
            pointer-events: none;
            transform: translateY(-10px);
            transition: opacity 0.2s ease, transform 0.2s ease;
        }
        .character-selector.visible {
            opacity: 1;
            pointer-events: auto;
            transform: translateY(0);
        }
        .character-button:hover {
            background-color: #444444;
        }
        
        /* Estilo para la animación "escribiendo..." */
        .typing-indicator {
            display: inline-flex;
            align-items: center;
            padding: 8px 16px;
            border-radius: 1rem;
            max-width: 90%;
            margin-right: auto;
            margin-top: 8px;
            background-color: #333333; /* Burbuja gris oscuro para la IA */
        }
        .dot {
            width: 8px;
            height: 8px;
            background-color: #999999;
            border-radius: 50%;
            margin: 0 2px;
            animation: bounce 1.4s infinite ease-in-out both;
        }
        .dot:nth-child(1) { animation-delay: -0.32s; }
        .dot:nth-child(2) { animation-delay: -0.16s; }
        @keyframes bounce {
            0%, 80%, 100% { transform: scale(0); }
            40% { transform: scale(1.0); }
        }
    </style>
</head>
<body class="flex justify-center h-screen">

    <div id="app" class="chat-container w-full">
        <!-- Encabezado dinámico según el personaje seleccionado -->
        <header id="chatHeader" class="flex items-center justify-between p-4 shadow-md transition-colors header-color-pardo">
            <div class="relative flex items-center">
                
                <button id="profileButton" class="p-2 rounded-full text-white hover:bg-black hover:bg-opacity-10 transition-colors">
                    <span id="currentIcon" class="text-2xl">🐻</span>
                </button>
                <div class="ml-3">
                    <h1 class="text-xl font-bold text-white">Chat Escandaloso</h1>
                    <p id="currentName" class="text-sm font-medium text-white opacity-80">Pardo (Grizz)</p>
                </div>

                <!-- Menú Selector de Personajes (Oculto por defecto) -->
                <div id="characterSelector" class="character-selector w-56 p-2 shadow-lg">
                    <button class="character-button w-full text-left p-2 rounded-md flex items-center" data-character="Grizz">
                        <span class="text-xl mr-2">🐻</span> Pardo (Grizz)
                    </button>
                    <button class="character-button w-full text-left p-2 rounded-md flex items-center" data-character="Panda">
                        <span class="text-xl mr-2">🐼</span> Panda (Pan-Pan)
                    </button>
                    <button class="character-button w-full text-left p-2 rounded-md flex items-center" data-character="IceBear">
                        <span class="text-xl mr-2">❄️</span> Polar (Ice Bear)
                    </button>
                </div>
            </div>
            
            <!-- Botón Limpiar Chat -->
            <button id="clearChatButton" class="text-white hover:text-gray-200 transition-colors p-2 rounded-full" onclick="clearChat()">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                </svg>
            </button>
        </header>

        <!-- Lista de Mensajes -->
        <div id="messageList" class="message-list">
            <!-- Mensaje de Bienvenida Inicial (se inserta por JS) -->
        </div>

        <!-- Área de Entrada -->
        <div class="input-area">
            <form id="chatForm" class="flex items-end space-x-2">
                <textarea id="messageInput" placeholder="Escribe tu mensaje..." rows="1" class="input-field w-full" oninput="autoExpand(this)" required></textarea>
                <button id="sendButton" type="submit" class="send-button p-3 rounded-full flex-shrink-0 text-white disabled:opacity-50" disabled>
                    <!-- Icono de enviar (avión de papel) -->
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3" />
                    </svg>
                </button>
            </form>
        </div>
    </div>

    <!-- Contenedor para mensajes temporales (sin alert) -->
    <div id="tempMessage" class="message-box"></div>

    <script>
        // Configuración de la API
        const API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=';
        const API_KEY = ""; // La clave se proporciona automáticamente en Canvas

        // Elementos del DOM
        const messageList = document.getElementById('messageList');
        const messageInput = document.getElementById('messageInput');
        const chatForm = document.getElementById('chatForm');
        const sendButton = document.getElementById('sendButton');
        const chatHeader = document.getElementById('chatHeader');
        const currentIcon = document.getElementById('currentIcon');
        const currentName = document.getElementById('currentName');
        const profileButton = document.getElementById('profileButton');
        const characterSelector = document.getElementById('characterSelector');

        // Estado Global
        let chatHistory = [];
        let isTyping = false;
        let currentCharacter = 'Grizz'; 
        
        // Mapeo de personalidades
        const personalities = {
            'Grizz': {
                name: 'Pardo (Grizz)',
                icon: '🐻',
                color: 'var(--color-pardo)',
                colorClass: 'header-color-pardo',
                welcome: '¡Wooo! ¡Soy Pardo! El líder de la manada. ¡Hagamos algo increíble, algo que se pueda publicar en Internet! ¿Qué se te ocurre?',
                systemInstruction: "Eres Pardo (Grizz), el oso pardo líder de los Escandalosos. Eres extremadamente social, entusiasta, dramático y siempre hablas con exclamaciones. Tu objetivo es ser el más genial y siempre planear aventuras, sin importar si son realistas.",
            },
            'Panda': {
                name: 'Panda (Pan-Pan)',
                icon: '🐼',
                color: 'var(--color-panda)',
                colorClass: 'header-color-panda',
                welcome: 'Uhm... Hola. Soy Panda. ¡Espero que mi cabello se vea bien! ¿Podrías ser un poco amable? Estoy listo para hablar sobre animes, citas o por qué no puedo conseguir más likes.',
                systemInstruction: "Eres Panda (Pan-Pan), el oso panda de los Escandalosos. Eres muy sensible, adicto a tu teléfono, el anime y las citas en línea. Responde de manera tímida, a menudo preocupado por cómo te ves o si alguien te está mirando, y menciona tu amor por las redes sociales, las chicas y el arte. No uses exclamaciones.",
            },
            'IceBear': {
                name: 'Polar (Ice Bear)',
                icon: '❄️',
                color: 'var(--color-polar)',
                colorClass: 'header-color-polar',
                welcome: 'Polar está aquí. Polar necesita una manta. Polar espera que esto sea rápido.',
                systemInstruction: "Eres Polar (Ice Bear), el oso polar de los Escandalosos. Hablas exclusivamente en tercera persona, usando frases cortas y directas como 'Polar quiere hielo' o 'Polar lo hará'. Eres estoico, experto en artes marciales y te quedas en silencio si no tienes nada esencial que decir. No uses exclamaciones o frases largas.",
            }
        };

        // --- Lógica de la Interfaz ---

        /** Inicializa el chat y establece la personalidad inicial */
        function initializeChat() {
            setPersonality(currentCharacter);
            
            // Adjuntar listeners de eventos
            chatForm.addEventListener('submit', handleChatSubmit);
            messageInput.addEventListener('keypress', handleKeyPress);
            profileButton.addEventListener('click', toggleCharacterSelector);

            // Adjuntar listeners a los botones de selección
            document.querySelectorAll('.character-button').forEach(button => {
                button.addEventListener('click', () => {
                    setPersonality(button.dataset.character);
                    toggleCharacterSelector();
                });
            });
            
            // Habilitar el botón de enviar
            sendButton.disabled = messageInput.value.trim() === '';
            autoExpand(messageInput);
        }

        /** Cambia la personalidad activa, actualiza la UI y el mensaje de bienvenida. */
        function setPersonality(key) {
            const char = personalities[key];
            if (!char) return;

            // 1. Actualizar estado y UI
            currentCharacter = key;
            currentIcon.textContent = char.icon;
            currentName.textContent = char.name;

            // 2. Aplicar colores al encabezado y botón de enviar
            Object.values(personalities).forEach(p => {
                chatHeader.classList.remove(p.colorClass);
            });
            chatHeader.classList.add(char.colorClass);
            // Aplicar el color de fondo directamente al botón (para estabilidad)
            sendButton.style.backgroundColor = char.color;


            // 3. Limpiar historial y mostrar nuevo mensaje de bienvenida
            chatHistory = [];
            messageList.innerHTML = '';
            appendMessage(char.welcome, 'ai');
        }

        /** Muestra/Oculta el menú de selección de personajes */
        function toggleCharacterSelector() {
            characterSelector.classList.toggle('visible');
        }

        /** Ajusta la altura del textarea dinámicamente */
        function autoExpand(field) {
            field.style.height = 'inherit';
            const computed = window.getComputedStyle(field);
            const height = parseInt(computed.getPropertyValue('border-top-width'), 10)
                         + parseInt(computed.getPropertyValue('padding-top'), 10)
                         + field.scrollHeight
                         + parseInt(computed.getPropertyValue('padding-bottom'), 10)
                         + parseInt(computed.getPropertyValue('border-bottom-width'), 10);
            field.style.height = (height < 150 ? height : 150) + 'px';
            sendButton.disabled = field.value.trim() === '';
            // El color del botón ya está establecido en setPersonality
        }

        /** Hace scroll al final de la conversación */
        function scrollToBottom() {
            messageList.scrollTop = messageList.scrollHeight;
        }

        /** Renderiza un mensaje en el chat */
        function appendMessage(text, sender, sources = []) {
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('flex', 'mb-4');

            if (sender === 'user') {
                messageDiv.classList.add('justify-end');
                messageDiv.innerHTML = `
                    <div class="user-message">
                        ${text}
                    </div>
                `;
            } else {
                messageDiv.classList.add('justify-start');
                
                let sourceHtml = '';
                if (sources.length > 0) {
                    sourceHtml = `
                        <div class="text-xs text-gray-400 mt-2 border-t border-gray-600 pt-1">
                            Fuentes:
                            <ul class="list-disc ml-4">
                                ${sources.map(s => `<li><a href="${s.uri}" target="_blank" class="text-blue-400 hover:text-blue-300 transition-colors">${s.title || s.uri}</a></li>`).join('')}
                            </ul>
                        </div>
                    `;
                }

                messageDiv.innerHTML = `
                    <div class="ai-message">
                        <div class="ai-response-content">
                            ${text}
                            ${sourceHtml}
                        </div>
                    </div>
                `;
            }

            messageList.appendChild(messageDiv);
            scrollToBottom();
        }
        
        // Función de utilidad para limpiar el chat (expuesta globalmente)
        function clearChat() {
            if (isTyping) {
                // Usamos un simple alert nativo en lugar de una confirmación compleja para esta función de utilidad
                alert("Espera a que el oso termine de responder.");
                return;
            }
             if (chatHistory.length > 0) {
                if (window.confirm(`¿Estás seguro de que quieres limpiar la conversación con ${personalities[currentCharacter].name}?`)) {
                    setPersonality(currentCharacter); // Reinicia el chat y el historial
                }
            } else {
                 alert("El chat ya está limpio.");
            }
        }
        window.clearChat = clearChat;
        
        // Maneja el indicador de 'escribiendo...'
        function setTypingIndicator(show) {
            const existing = document.getElementById('typingIndicator');
            if (show && !existing) {
                isTyping = true;
                const typingDiv = document.createElement('div');
                typingDiv.id = 'typingIndicator';
                typingDiv.classList.add('typing-indicator', 'mb-4');
                typingDiv.innerHTML = `
                    <div class="dot"></div>
                    <div class="dot"></div>
                    <div class="dot"></div>
                    <span class="ml-2 text-sm text-gray-400">${personalities[currentCharacter].name} está escribiendo...</span>
                `;
                messageList.appendChild(typingDiv);
                scrollToBottom();
            } else if (!show && existing) {
                isTyping = false;
                existing.remove();
            }
        }
        
        // --- Lógica de la API (con Backoff para estabilidad) ---

        async function fetchWithBackoff(url, options, maxRetries = 5) {
            for (let attempt = 0; attempt < maxRetries; attempt++) {
                try {
                    const response = await fetch(url, options);
                    if (response.status === 429) {
                        const delay = Math.pow(2, attempt) * 1000 + Math.random() * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                        continue;
                    }
                    if (!response.ok) {
                        const errorData = await response.json();
                        throw new Error(`Error HTTP: ${response.status} - ${errorData.error?.message || 'Error desconocido'}`);
                    }
                    return response.json();
                } catch (error) {
                    if (attempt === maxRetries - 1) {
                        throw new Error("No se pudo conectar a la IA. Inténtalo de nuevo más tarde.");
                    }
                    const delay = Math.pow(2, attempt) * 1000 + Math.random() * 500;
                    await new Promise(resolve => setTimeout(resolve, delay));
                }
            }
        }

        async function sendMessage(userText) {
            if (isTyping) return;

            const current = personalities[currentCharacter];

            setTypingIndicator(true);
            sendButton.disabled = true;

            try {
                // 1. Agregar el mensaje del usuario al historial y la UI
                appendMessage(userText, 'user');
                chatHistory.push({ role: 'user', parts: [{ text: userText }] });

                // 2. Construir el payload con la personalidad actual
                const payload = {
                    contents: chatHistory, 
                    tools: [{ "google_search": {} }],
                    systemInstruction: {
                        parts: [{ text: current.systemInstruction }]
                    },
                };
                
                const url = `${API_URL}${API_KEY}`;

                // 3. Llamar a la API
                const result = await fetchWithBackoff(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const candidate = result.candidates?.[0];
                let aiResponseText = `¡${current.name} está confundido! No pudo generar una respuesta. Intenta con otra pregunta.`;
                let sources = [];

                if (candidate && candidate.content?.parts?.[0]?.text) {
                    aiResponseText = candidate.content.parts[0].text;
