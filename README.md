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

![image](https://github.com/user-attachments/assets/38744a79-732c-4eab-941f-fd151313d9c7)

![image](https://github.com/user-attachments/assets/c92a4b39-b852-48d1-9c7d-218d08714b6d)


```HTML & PHP
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voice Recorder</title>
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
        #transcript {
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
    </div>

    <script>
        const startBtn = document.getElementById('startBtn');
        const stopBtn = document.getElementById('stopBtn');
        const transcriptDiv = document.getElementById('transcript');

        let recognition;
        if ('webkitSpeechRecognition' in window) {
            recognition = new webkitSpeechRecognition();
        } else if ('SpeechRecognition' in window) {
            recognition = new SpeechRecognition();
        } else {
            alert('Your browser does not support speech recognition.');
        }

        let debounceTimeout;
        let lastSentText = '';

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

                if (debounceTimeout) {
                    clearTimeout(debounceTimeout);
                }

                debounceTimeout = setTimeout(() => {
                    if (transcript !== lastSentText) {
                        sendToAPI(transcript);
                        lastSentText = transcript;
                    }
                }, 500);
            };

            startBtn.addEventListener('click', () => {
                recognition.start();
                transcriptDiv.textContent = 'Listening...';
            });

            stopBtn.addEventListener('click', () => {
                recognition.stop();
                transcriptDiv.textContent = 'Stopped.';
            });
        }

        function sendToAPI(text) {
            fetch(window.location.href, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ prompt: text })
            })
            .then(response => response.json())
            .then(data => {
                alert(data.response);
            })
            .catch(error => {
                console.error('Error:', error);
            });
        }
    </script>

    <?php
    if ($_SERVER['REQUEST_METHOD'] == 'POST') {
        header('Content-Type: application/json');

        // Database configuration
        $servername = "localhost";
        $username = "root"; // replace with your database username
        $password = ""; // replace with your database password
        $dbname = "robotcontrol";

        // Create connection
        $conn = new mysqli($servername, $username, $password, $dbname);

        // Check connection
        if ($conn->connect_error) {
            die(json_encode(['response' => 'Connection failed: ' . $conn->connect_error]));
        }

        // Get the JSON input
        $data = json_decode(file_get_contents('php://input'), true);
        $prompt = $data['prompt'];

        // Insert into the database
        $sql = "INSERT INTO api (Text) VALUES ('$prompt')";
        if ($conn->query($sql) === TRUE) {
            $response = ['response' => 'Text saved successfully!'];
        } else {
            $response = ['response' => 'Error: ' . $sql . '<br>' . $conn->error];
        }

        $conn->close();

        // Return response
        echo json_encode($response);
        exit();
    }
    ?>
</body>
</html>
