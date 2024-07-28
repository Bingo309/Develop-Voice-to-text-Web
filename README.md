# Develop-Voice-to-text-Web

# Server:
Utilized XAMPP as the server software to host the website locally during development

#API Integration:

Integrated a voice-to-text conversion API to enable users to upload or record audio files
The API would then transcribe the audio into text

# Database (phpmyadmin):
Implemented a database to store user transcribed text.

# Languages Used:
-HTML
-PHP

# Website
![image](https://github.com/user-attachments/assets/208632ed-2638-4c34-ac40-285159683eca)

![image](https://github.com/user-attachments/assets/769bbb8c-8d11-4d8c-b657-0b3622612143)


```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voice Recorder with AI Response</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
        }
        .container {
            text-align: center;
        }
        button {
            margin: 10px;
            padding: 10px 20px;
            font-size: 16px;
        }
        #transcript, #aiResponse {
            margin-top: 20px;
            font-size: 18px;
            color: #333;
        }
    </style>
</head>
<body>
    <div class="container">
        <button id="startBtn">Start Record</button>
        <button id="stopBtn">Stop Record</button>
        <div id="transcript"></div>
        <div id="aiResponse"></div>
    </div>

    <script>
        const startBtn = document.getElementById('startBtn');
        const stopBtn = document.getElementById('stopBtn');
        const transcriptDiv = document.getElementById('transcript');
        const aiResponseDiv = document.getElementById('aiResponse');

        let recognition;
        if ('webkitSpeechRecognition' in window) {
            recognition = new webkitSpeechRecognition();
        } else if ('SpeechRecognition' in window) {
            recognition = new SpeechRecognition();
        } else {
            alert('Your browser does not support speech recognition.');
        }

        if (recognition) {
            recognition.continuous = true;
            recognition.interimResults = true;
            recognition.lang = 'en-US';

            recognition.onresult = (event) => {
                let transcript = '';
                for (let i = event.resultIndex; i < event.results.length; i++) {
                    transcript += event.results[i][0].transcript;
                }
                transcriptDiv.textContent = transcript;
                sendToAPI(transcript);
            };

            startBtn.addEventListener('click', () => {
                recognition.start();
                transcriptDiv.textContent = 'Listening...';
                aiResponseDiv.textContent = '';
            });

            stopBtn.addEventListener('click', () => {
                recognition.stop();
                transcriptDiv.textContent = 'Stopped.';
            });
        }

        function sendToAPI(text) {
            fetch('/api/generate', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ prompt: text })
            })
            .then(response => response.json())
            .then(data => {
                aiResponseDiv.textContent = data.response;
            })
            .catch(error => {
                console.error('Error:', error);
            });
        }
    </script>
</body>
</html>
