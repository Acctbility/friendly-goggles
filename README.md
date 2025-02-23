
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accountability App</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 0; }
        .container { max-width: 600px; margin: 20px auto; padding: 20px; border: 1px solid #ddd; border-radius: 8px; }
        h2 { font-size: 24px; margin-bottom: 16px; }
        img { max-width: 100%; margin-bottom: 16px; }
        textarea { width: 100%; padding: 8px; margin-bottom: 16px; border: 1px solid #ddd; border-radius: 4px; }
        button { width: 100%; padding: 10px; background-color: #4CAF50; color: white; border: none; border-radius: 4px; cursor: pointer; }
        button:hover { background-color: #45a049; }
    </style>
</head>
<body>
    <div class="container">
        <h2>Accountability</h2>
        <button onclick="captureImage()">Capture Image</button>
        <br><br>
        <img id="capturedImage" alt="Captured Image">
        <textarea id="comment" placeholder="Describe the issue..."></textarea>
        <button onclick="handleSubmit()">Submit</button>
    </div>

    <script>
        let imageSrc = null;
        let locationData = null;

        navigator.geolocation.getCurrentPosition(position => {
            locationData = {
                latitude: position.coords.latitude,
                longitude: position.coords.longitude,
                timestamp: new Date(position.timestamp).toLocaleString()
            };
        });

        async function captureImage() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                const video = document.createElement('video');
                video.srcObject = stream;
                await video.play();

                const canvas = document.createElement('canvas');
                canvas.width = 640;
                canvas.height = 480;
                const context = canvas.getContext('2d');
                context.drawImage(video, 0, 0, canvas.width, canvas.height);
                imageSrc = canvas.toDataURL('image/png');
                document.getElementById('capturedImage').src = imageSrc;

                stream.getTracks().forEach(track => track.stop());
            } catch (error) {
                console.error('Failed to capture image', error);
            }
        }

        async function handleSubmit() {
            const comment = document.getElementById('comment').value;
            if (!imageSrc || !locationData) {
                alert('Please capture an image with geolocation enabled.');
                return;
            }

            const tweetContent = `Citizen Report: ${comment}. Location: Lat ${locationData.latitude}, Long ${locationData.longitude}. Timestamp: ${locationData.timestamp}. @Acctbility`;

            try {
                const response = await fetch('https://api.twitter.com/2/tweets', {
                    method: 'POST',
                    headers: {
                        'Authorization': 'AAAAAAAAAAAAAAAAAAAAAIf%2FzQEAAAAAQ%2F5EkQgB6k2IC1mEnTOVRL9kjxI%3DUVbMtnyPkdlGSfa1j7HM50awoDqceg90YvuwjXhlKh2npyUTkl',
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        "text": tweetContent
                    })
                });

                if (response.ok) {
                    alert('Tweet posted successfully!');
                } else {
                    console.error('Failed to post tweet', response.statusText);
                    alert('Failed to post tweet. Check console for details.');
                }
            } catch (error) {
                console.error('Error posting to Twitter', error);
                alert('Error posting to Twitter. Check console for details.');
            }
        }

    </script>
</body>
</html>
    
