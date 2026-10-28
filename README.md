<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Review Card Router</title>
    <style>
        body { font-family: sans-serif; background: #f4f7f6; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
        .box { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); max-width: 350px; text-align: center; }
        h2 { color: #1a73e8; }
        .btn { display: inline-block; background: #1a73e8; color: white; padding: 12px 24px; text-decoration: none; border-radius: 6px; font-weight: bold; margin-top: 15px; }
        .hidden { display: none; }
    </style>
</head>
<body>

    <div class="box">
        <div id="loading">Checking activation status...</div>

        <div id="activation-box" class="hidden">
            <h2>Setup Review Card</h2>
            <p>Card ID: <strong id="display-id">---</strong> is not activated yet.</p>
            <p>Click below to link this card to your Google Business Profile.</p>
            <a id="form-link" href="#" target="_blank" class="btn">Activate Card Now</a>
        </div>

        <div id="error-box" class="hidden">
            <h2>Error</h2>
            <p>Invalid card scan. Missing ID parameter.</p>
        </div>
    </div>

    <script>
        const GOOGLE_SHEET_CSV = "https://google.com";
        const GOOGLE_FORM_URL = "https://docs.google.com/forms/d/e/1FAIpQLSeWxhPnYOh5BP4wbo49iXlJ5Esx_oQ4wzQN0L6qmf_Ebohs8A/viewform";

        const urlParams = new URLSearchParams(window.location.search);
        const cardId = urlParams.get('id');

        if (!cardId) {
            document.getElementById('loading').classList.add('hidden');
            document.getElementById('error-box').classList.remove('hidden');
        } else {
            document.getElementById('display-id').textContent = cardId;
            document.getElementById('form-link').href = `${GOOGLE_FORM_URL}?usp=pp_url&entry.1226768391=${encodeURIComponent(cardId)}`;
            checkSheet();
        }

        async function checkSheet() {
            try {
                const res = await fetch(GOOGLE_SHEET_CSV);
                const text = await res.text();
                const lines = text.split(/\r?\n/);
                
                let redirectUrl = "";

                // Search form responses from bottom to top (get the newest update)
                for (let i = lines.length - 1; i >= 1; i--) {
                    if (!lines[i]) continue;
                    const cols = lines[i].split(',');
                    if (cols[1] && cols[1].replace(/"/g, '').trim() === cardId.trim()) {
                        redirectUrl = cols[2] ? cols[2].replace(/"/g, '').trim() : "";
                        break;
                    }
                }

                if (redirectUrl && redirectUrl.toLowerCase().startsWith("http")) {
                    window.location.href = redirectUrl;
                } else {
                    document.getElementById('loading').classList.add('hidden');
                    document.getElementById('activation-box').classList.remove('hidden');
                }
            } catch (err) {
                document.getElementById('loading').classList.add('hidden');
                document.getElementById('activation-box').classList.remove('hidden');
            }
        }
    </script>
</body>
</html>
