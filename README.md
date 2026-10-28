<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Review Card Router</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }
        .container {
            background: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
            max-width: 400px;
            width: 100%;
            text-align: center;
        }
        h2 { color: #1a73e8; margin-top: 0; }
        p { font-size: 15px; line-height: 1.5; color: #5f6368; }
        .btn {
            display: inline-block;
            background-color: #1a73e8;
            color: white;
            padding: 12px 24px;
            text-decoration: none;
            border-radius: 6px;
            font-weight: 500;
            margin-top: 15px;
            transition: background 0.2s;
        }
        .btn:hover { background-color: #1557b0; }
        #loading { font-weight: bold; color: #1a73e8; }
        .hidden { display: none; }
    </style>
</head>
<body>

    <div class="container">
        <!-- Checking Status -->
        <div id="loading">
            <p>Checking card activation status...</p>
        </div>

        <!-- Unclaimed Card State -->
        <div id="activation-box" class="hidden">
            <h2>Setup Review Card</h2>
            <p>Card ID: <strong id="display-id" style="color:#1a73e8;">---</strong> is not activated yet.</p>
            <p>If you purchased this card, click below to link it to your Google Business Profile instantly.</p>
            <a id="form-link" href="#" target="_blank" class="btn">Configure Card Now</a>
        </div>

        <!-- Error State -->
        <div id="error-box" class="hidden">
            <h2>No Card ID Detected</h2>
            <p>Please scan a physical QR code card containing a valid identity parameter.</p>
        </div>
    </div>

    <script>
        // Your Exact Live Google CSV Data Link 
        const GOOGLE_SHEET_CSV_URL = "https://google.com";
        const formBaseUrl = "https://google.com";

        const urlParams = new URLSearchParams(window.location.search);
        const cardId = urlParams.get('id');

        const loadingDiv = document.getElementById('loading');
        const activationBox = document.getElementById('activation-box');
        const errorBox = document.getElementById('error-box');
        const displayId = document.getElementById('display-id');
        const formLink = document.getElementById('form-link');

        if (!cardId) {
            loadingDiv.classList.add('hidden');
            errorBox.classList.remove('hidden');
        } else {
            const cleanId = cardId.trim();
            displayId.textContent = cleanId;
            
            // Auto-fills the Card ID field in your Google Form
            formLink.href = `${formBaseUrl}?usp=pp_url&entry.1226768391=${encodeURIComponent(cleanId)}`;
            
            checkDatabase(cleanId);
        }

        async function checkDatabase(targetId) {
            try {
                const response = await fetch(GOOGLE_SHEET_CSV_URL);
                if (!response.ok) throw new Error("Network offline");
                const csvText = await response.text();
                
                const lines = csvText.split(/\r?\n/);
                
                let isRegistered = false;
                let finalDestination = "";

                // Read from right to left (bottom to top of your form responses) 
                // This ensures if a shop updates their link later, the newest link works!
                for (let i = lines.length - 1; i >= 1; i--) {
                    if (!lines[i]) continue;
                    
                    const columns = lines[i].split(',');
                    if (columns.length >= 3) {
                        // Matching Columns: [0]=Timestamp, [1]=Card ID, [2]=Review Link
                        const currentCardId = columns[1] ? columns[1].replace(/"/g, '').trim() : "";
                        const currentUrl = columns[2] ? columns[2].replace(/"/g, '').trim() : "";

                        if (currentCardId === targetId && currentUrl.toLowerCase().startsWith("http")) {
                            isRegistered = true;
                            finalDestination = currentUrl;
                            break; // Found the most recent active entry
                        }
                    }
                }

                if (isRegistered && finalDestination) {
                    window.location.href = finalDestination;
                } else {
                    loadingDiv.classList.add('hidden');
                    activationBox.classList.remove('hidden');
                }

            } catch (err) {
                console.error("Database connection failure:", err);
                loadingDiv.classList.add('hidden');
                activationBox.classList.remove('hidden');
            }
        }
    </script>
</body>
</html>
