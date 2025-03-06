#Conversational-AI-Chatbot
<!DOCTYPE html>
<html>
<head>
    <title>KRISHNA's Jarvis</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #1A2980, #26D0CE);
            color: #ecf0f1;
            overflow: hidden;
            position: relative;
        }

        #chat-container {
            flex-grow: 1;
            overflow-y: auto;
            padding: 30px;
            border-radius: 15px;
            margin: 30px;
            background-color: rgba(0, 0, 0, 0.4);
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
            backdrop-filter: blur(10px);
        }

        #input-container {
            display: flex;
            padding: 20px;
            border-top: 1px solid rgba(255, 255, 255, 0.1);
            margin: 0 30px 30px 30px;
            background-color: rgba(41, 128, 185, 0.3);
            border-radius: 0 0 15px 15px;
        }

        #message-input {
            flex-grow: 1;
            padding: 15px 20px;
            border: none;
            border-radius: 30px;
            font-size: 18px;
            background-color: rgba(255, 255, 255, 0.1);
            color: #ecf0f1;
            outline: none;
            transition: background-color 0.3s ease, border-color 0.3s ease;
        }

        #message-input:focus {
            background-color: rgba(255, 255, 255, 0.2);
        }

        #send-button {
            padding: 15px 30px;
            background: linear-gradient(to right, #3498db, #2980b9);
            color: white;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            margin-left: 20px;
            font-size: 18px;
            transition: background 0.3s ease, transform 0.2s ease;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        }

        #send-button:hover {
            background: linear-gradient(to right, #2980b9, #3498db);
            transform: translateY(-2px);
        }

        .message {
            padding: 20px;
            margin-bottom: 20px;
            border-radius: 25px;
            max-width: 85%;
            word-wrap: break-word;
            box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
        }

        .user {
            background-color: #34495e;
            align-self: flex-end;
            color: #bdc3c7;
        }

        .assistant {
            background-color: #2c3e50;
            color: #ecf0f1;
        }

        #loading-indicator {
            text-align: center;
            margin-top: 30px;
            display: none;
        }

        #loading-indicator img {
            height: 50px;
            animation: spin 2s linear infinite;
        }

        @keyframes spin {
            100% {
                transform: rotate(360deg);
            }
        }

        #hi {
            color: #3498db;
            font-family: 'Courier New', Courier, monospace;
            text-align: center;
            margin: 30px 0;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.6);
            font-size: 2.5em;
        }

        .assistant pre {
            background-color: #34495e;
            padding: 20px;
            overflow-x: auto;
            border-radius: 12px;
            line-height: 1.7;
            white-space: pre-wrap;
        }

        .assistant ul, .assistant ol {
            padding-left: 30px;
            margin-top: 15px;
            line-height: 1.7;
        }

        .assistant blockquote {
            border-left: 5px solid #3498db;
            padding-left: 20px;
            margin: 20px 0;
            font-style: italic;
            color: #95a5a6;
            line-height: 1.7;
        }

        html {
            scroll-behavior: smooth;
        }

        #logo-section {
            position: absolute;
            top: 20px; /* Changed to top */
            right: 20px; /* Changed to right */
            display: flex;
            align-items: center;
            color: #bdc3c7;
        }

        #logo-section img {
            height: 100px;
            margin-right: 10px;
        }
    </style>
</head>
<body>
    <h1 id="hi">Jarvis</h1>
    <div id="chat-container"></div>
    <div id="loading-indicator"><img src="loading.gif" alt="Loading..."></div>
    <div id="input-container">
        <input type="text" id="message-input" placeholder="Type your message...">
        <button id="send-button">Send</button>
    </div>

    <div id="logo-section">
        <img src="c:\Users\krishnamurthy.goggi\Downloads\logo-web.png" alt="Logo">

    <script>
        const chatContainer = document.getElementById("chat-container");
        const messageInput = document.getElementById("message-input");
        const sendButton = document.getElementById("send-button");
        const loadingIndicator = document.getElementById("loading-indicator");
        const apiKey = "ADD API KEY HERE"; // VERY IMPORTANT: REPLACE WITH YOUR GEMINI API KEY

        sendButton.addEventListener("click", sendMessage);
        messageInput.addEventListener("keydown", (event) => {
            if (event.key === "Enter") {
                sendMessage();
            }
        });

        function sendMessage() {
            const message = messageInput.value.trim();
            if (message === "") return;

            appendMessage("user", message);
            messageInput.value = "";
            loadingIndicator.style.display = "block";

            fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: message
                        }]
                    }]
                }),
            })
            .then((response) => response.json())
            .then((data) => {
                loadingIndicator.style.display = "none";
                if (data.candidates && data.candidates.length > 0 && data.candidates[0].content && data.candidates[0].content.parts && data.candidates[0].content.parts.length > 0) {
                    let responseText = data.candidates[0].content.parts[0].text;
                    responseText = formatResponse(responseText);
                    appendMessage("assistant", responseText);
                } else {
                    appendMessage("assistant", "Error: No response from Gemini.");
                }
            })
            .catch((error) => {
                loadingIndicator.style.display = "none";
                console.error("Error:", error);
                appendMessage("assistant", "Sorry, an error occurred.");
            });
        }

        function appendMessage(sender, message) {
            const messageDiv = document.createElement("div");
            messageDiv.classList.add("message", sender);
            messageDiv.innerHTML = message;
            chatContainer.appendChild(messageDiv);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function formatResponse(text) {
            text = text.replace(/```([\s\S]*?)```/g, '<pre>$1</pre>');
            text = text.replace(/\*\*(.*?)\*\*/g, '<b>$1</b>');
            text = text.replace(/\*(.*?)\*/g, '<i>$1</i>');
            text = text.replace(/^(#+)\s+(.+)$/gm, (match, hashes, content) => {
                const level = hashes.length;
                return `<h${level}>${content}</h${level}>`;
            });
            text = text.replace(/^> (.*)$/gm, '<blockquote>$1</blockquote>');
            text = text.replace(/^-\s+(.*)$/gm, '<li>$1</li>');
            text = text.replace(/^\d+\.\s+(.*)$/gm, '<li>$1</li>');

            const listRegex = /(?:^|\n)([-*]|\d+\.)\s+.+(?:\n(?:[-*]|\d+\.).+)*/g;
            text = text.replace(listRegex, (match) => {
                const lines = match.trim().split('\n');
                let listItems = '';
                lines.forEach(line => {
                    listItems += `<li>${line.replace(/^[-*\d+\.]\s+/, '')}</li>`;
                });
                if (lines[0].startsWith('-') || lines[0].startsWith('*')) {
                    return `<ul>${listItems}</ul>`;
                } else {
                    return `<ol>${listItems}</ol>`;
                }
            });

            return text;
        }
    </script>
</body>
</html>
