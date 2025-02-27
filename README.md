
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <title>Speech Live Translation</title>
    <!-- Load the Speech SDK from the CDN -->
    <script src="https://aka.ms/csspeech/jsbrowserpackageraw"></script>
    <style>
        html,
        body {
            margin: 0;
            padding: 0;
            height: 100%;
            background: #000;
            font-family: sans-serif;
            color: #fff;
        }

        nav {
            background: #000;
            padding: 1em;
            text-align: center;
        }

        nav h1 {
            margin: 0;
            font-size: 2rem;
            color: #fff;
        }

        #controlPanel {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 1em;
            margin-top: 1em;
        }

        #startBtn,
        #stopBtn,
        #fontSizeSelect,
        #sourceLangSelect,
        #targetLangSelect {
            padding: 0.5em 1em;
            font-size: 1rem;
            cursor: pointer;
            margin: 5px;
        }

        /* Fixed height with auto-scroll for a transcript view */
        #subtitleContainer {
            font-weight: bold;
            padding: 1em;
            margin-top: 1em;
            border: 1px solid #666;
            background: #000;
            color: #eee;
            text-align: left;
            box-sizing: border-box;
            line-height: 1.5em;
            max-height: 5.5em;
            overflow-y: auto;
            white-space: pre-wrap;
        }

        .controlls {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .controll_wrap {
            display: flex;
            align-items: flex-start;
            justify-content: space-evenly;
        }

        .playpausebtn {
            display: flex;

        }
    </style>
</head>

<body>
    <nav>
        <h1>Live Audio Translation</h1>
    </nav>
    <div id="subtitleContainer">
        <!-- Subtitles will appear here -->
    </div>

    <div class="controll_wrap">
        <div class="controlls">
            <h2>Controls</h2>
            <div class="playpausebtn">
                <button id="startBtn">Start</button>
                <button id="stopBtn">Stop</button>
            </div>
        </div>

        <div class="controlls">
            <h2>Font Size</h2>
            <select id="fontSizeSelect">
                <option value="2rem">Small</option>
                <option value="3rem" selected>Medium</option>
                <option value="4rem">Large</option>
                <option value="5rem">Extra Large</option>
            </select>
        </div>

        <div class="controlls">
            <h2>Source Language</h2>
            <select id="sourceLangSelect">
                <option value="en-US" selected>English</option>
                <option value="mr-IN">Marathi</option>
                <option value="hi-IN">Hindi</option>
                <option value="es-ES">Spanish</option>

            </select>
        </div>

        <div class="controlls">
            <h2>Target Language</h2>
            <select id="targetLangSelect">
                <option value="mr-IN" selected>Marathi</option>
                <option value="hi-IN">Hindi</option>
                <option value="en-US">English</option>
                <option value="es-ES">Spanish</option>
            </select>
        </div>
    </div>
    <script>
        const subscriptionKey = "3qNiyYidZAxNocF6edNPK3GdbBcn6JUAbwGDVl5Xl2I4mwfstbvmJQQJ99BBACGhslBXJ3w3AAAYACOGASO5";
        const serviceRegion = "centralindia";

        let translationRecognizer;
        let currentLineElement = null;

        const startBtn = document.getElementById("startBtn");
        const stopBtn = document.getElementById("stopBtn");
        const fontSizeSelect = document.getElementById("fontSizeSelect");
        const subtitleContainer = document.getElementById("subtitleContainer");
        const sourceLangSelect = document.getElementById("sourceLangSelect");
        const targetLangSelect = document.getElementById("targetLangSelect");

        startBtn.addEventListener("click", startTranslation);
        stopBtn.addEventListener("click", stopTranslation);

        fontSizeSelect.addEventListener("change", (e) => {
            subtitleContainer.style.fontSize = e.target.value;
        });

        function startTranslation() {
            const translationConfig = SpeechSDK.SpeechTranslationConfig
                .fromSubscription(subscriptionKey, serviceRegion);

            const sourceLang = sourceLangSelect.value;
            const targetLang = targetLangSelect.value;

            translationConfig.speechRecognitionLanguage = sourceLang;
            translationConfig.addTargetLanguage(targetLang);

            const audioConfig = SpeechSDK.AudioConfig.fromDefaultMicrophoneInput();
            translationRecognizer = new SpeechSDK.TranslationRecognizer(translationConfig, audioConfig);


            const languageMap = {
                "en-US": "en",
                "hi-IN": "hi",
                "mr-IN": "mr",
                "es-ES": "es"
            };

            translationRecognizer.recognizing = (s, e) => {
                if (e.result.reason === SpeechSDK.ResultReason.TranslatingSpeech) {
                    const shortTarget = languageMap[targetLang] || "en";
                    const partial = e.result.translations.get(shortTarget);
                    if (partial) {
                        if (!currentLineElement) {
                            currentLineElement = document.createElement("div");
                            subtitleContainer.appendChild(currentLineElement);
                        }
                        currentLineElement.textContent = partial + " ...";
                        subtitleContainer.scrollTop = subtitleContainer.scrollHeight;
                    }
                }
            };

            translationRecognizer.recognized = (s, e) => {
                if (e.result.reason === SpeechSDK.ResultReason.TranslatedSpeech) {
                    const shortTarget = languageMap[targetLang] || "en";
                    const finalTranslation = e.result.translations.get(shortTarget);
                    if (finalTranslation) {
                        if (!currentLineElement) {
                            currentLineElement = document.createElement("div");
                            subtitleContainer.appendChild(currentLineElement);
                        }
                        currentLineElement.textContent = finalTranslation;
                        subtitleContainer.scrollTop = subtitleContainer.scrollHeight;
                        currentLineElement = null;
                    }
                }
            };

            translationRecognizer.canceled = (s, e) => {
                console.log("CANCELED: Reason=", e.reason);
                if (e.reason === SpeechSDK.CancellationReason.Error) {
                    console.log("CANCELED: ErrorCode=", e.errorCode);
                    console.log("CANCELED: ErrorDetails=", e.errorDetails);
                }
                stopTranslation();
            };

            translationRecognizer.sessionStarted = (s, e) => {
                console.log("Session started:", e.sessionId);
            };

            translationRecognizer.sessionStopped = (s, e) => {
                console.log("Session stopped:", e.sessionId);
            };

            translationRecognizer.startContinuousRecognitionAsync(
                () => {
                    const newLine = document.createElement("div");
                    newLine.textContent = "listening.......";
                    subtitleContainer.appendChild(newLine);
                    subtitleContainer.scrollTop = subtitleContainer.scrollHeight;
                },
                err => {
                    console.log("Error: " + err);
                }
            );
        }

        function stopTranslation() {
            if (translationRecognizer) {
                translationRecognizer.stopContinuousRecognitionAsync(
                    () => {
                        console.log("Recognition stopped.");
                        translationRecognizer.close();
                        translationRecognizer = undefined;
                        const newLine = document.createElement("div");
                        newLine.textContent = "stopped....";
                        subtitleContainer.appendChild(newLine);
                        subtitleContainer.scrollTop = subtitleContainer.scrollHeight;
                    },
                    err => {
                        console.error(err);
                    }
                );
            }
        }
    </script>
</body>

</html>
