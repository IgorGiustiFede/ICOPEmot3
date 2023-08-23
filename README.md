# ICOPE
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>🎙️</text></svg>" />
    <title>Record and Send Voice Memo</title>
    <style>
        #recordBtn {
            background-color: blue;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            display: flex;
            align-items: center;
        }

        #recordBtn.recording {
            background-color: red;
        }

        #recordIcon {
            margin-right: 10px;
        }
    </style>
</head>
<body>
    <div>
        <h2>Record a voice memo</h2>
        <button id="recordBtn">
            <span id="recordIcon">🎙️</span>
            Record
        </button>
    </div>
</body>

<script>
    const recordBtn = document.getElementById('recordBtn');
    const audioChunks = [];
    let recorder;
    let audioStream;

    function startRecording() {
        navigator.mediaDevices.getUserMedia({ audio: true })
            .then(stream => {
                audioStream = stream;
                recorder = new MediaRecorder(stream);

                recorder.addEventListener('dataavailable', (event) => {
                    audioChunks.push(event.data);
                });

                recorder.addEventListener('stop', () => {
                    const audioBlob = new Blob(audioChunks, { type: 'audio/mp3' });
                    const fileName = generateFileName();
                    sendAudioData(audioBlob, fileName);
                });

                recorder.start();
                recordBtn.classList.add('recording');
            })
            .catch(err => console.error(err));
    }

    function stopRecording() {
        recorder.stop();
        audioStream.getAudioTracks().forEach(track => track.stop());
        recordBtn.classList.remove('recording');
    }

    function sendAudioData(data, fileName) {
        const serverUrl = "https://hook.eu1.make.com/1y1ku32ezig5rezo6ab995olarcijhcx";
        const formData = new FormData();
        formData.set('file', data, fileName);
        fetch(serverUrl, { method: "POST", body: formData })
            .then(response => console.log(`Audio data (${fileName}) sent to server.`))
            .catch(error => console.error("Error sending audio data to server: ", error));
    }

    function generateFileName() {
        const date = new Date();
        const year = date.getFullYear();
        const month = (date.getMonth() + 1).toString().padStart(2, '0');
        const day = date.getDate().toString().padStart(2, '0');
        const timestamp = date.getTime().toString();
        return `${year}-${month}-${day}-${timestamp}.mp3`;
    }

    recordBtn.addEventListener('click', () => {
        if (recorder && recorder.state === 'recording') {
            stopRecording();
        } else {
            startRecording();
        }
    });
</script>
</html>

