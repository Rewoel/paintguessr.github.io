<!DOCTYPE html>
<html>
<head>
    <title>Paint Guessr</title>
    <style>
        body {
            font-family: monospace;
            background-color: black;
            color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
            text-decoration: underline;
            transition: opacity 0.5s ease-in-out;
        }
        p {
            text-align: center;
            transition: opacity 0.5s ease-in-out;
        }
        .hidden-header {
            opacity: 0;
        }
        canvas {
            border: 1px solid white;
            cursor: crosshair;
            transition: transform 0.5s ease-in-out;
        }
        .color-button {
            width: 30px;
            height: 30px;
            border: none;
            cursor: pointer;
        }
        #wordList {
            margin-top: 20px;
        }
        #wordList span {
            margin: 0 10px;
            font-size: 20px;
            cursor: pointer;
        }
        #selectedColor {
            width: 30px;
            height: 30px;
            border: 1px solid white;
            margin-top: 10px;
            margin-left: auto;
            margin-right: auto;
            display: block;
        }
        #selectedWord {
            margin-top: 20px;
            font-size: 24px;
            font-weight: bold;
        }
        .hidden {
            display: none;
        }
        #finishButton, #restartButton {
            transition: opacity 0.5s ease-in-out;
        }
        .hidden-button {
            opacity: 0;
        }
        #brushSizeSlider {
            width: 150px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <h1 id="header">Paint Guessr</h1>
    <p id="instruction">Zeichne eines der Wörter:</p>

    <canvas id="myCanvas" width="500" height="300"></canvas>
    <div id="tools">
        <button id="eraserButton">Radiergummi</button>
        <button id="fillButton">Hintergrund</button>
        <div id="colorPicker">
            <button class="color-button" style="background-color: red;" data-color="red"></button>
            <button class="color-button" style="background-color: green;" data-color="green"></button>
            <button class="color-button" style="background-color: blue;" data-color="blue"></button>
            <button class="color-button" style="background-color: yellow;" data-color="yellow"></button>
            <button class="color-button" style="background-color: white;" data-color="white"></button>
        </div>
        <input type="range" id="brushSizeSlider" min="1" max="20" value="5">
        <div id="selectedColor"></div>
    </div>

    <div id="wordList"></div>

    <div id="selectedWord"></div>

    <button id="finishButton">Fertig</button>
    <button id="restartButton" class="hidden-button">Neustart</button>

    <script>
        const canvas = document.getElementById("myCanvas");
        const ctx = canvas.getContext("2d");
        let isDrawing = false;
        let isErasing = false;
        let currentColor = "white";
        let drawingEnabled = true;
        let brushSize = 5;

        const brushSizeSlider = document.getElementById("brushSizeSlider");
        brushSizeSlider.addEventListener("input", () => {
            brushSize = parseInt(brushSizeSlider.value);
        });

        const eraserButton = document.getElementById("eraserButton");
        eraserButton.addEventListener("click", () => {
            isErasing = !isErasing;
        });

        const fillButton = document.getElementById("fillButton");
        fillButton.addEventListener("click", () => {
            fillCanvas();
        });

        const colorButtons = document.querySelectorAll(".color-button");
        colorButtons.forEach(button => {
            button.addEventListener("click", () => {
                currentColor = button.dataset.color;
                updateSelectedColor();
            });
        });

        const finishButton = document.getElementById("finishButton");
        finishButton.addEventListener("click", () => {
            drawingEnabled = false;
            document.getElementById("tools").classList.add("hidden");
            canvas.style.transform = "scale(1.5)";
            document.getElementById("selectedWord").style.transform = "scale(1.5)";
            document.getElementById("header").classList.add("hidden-header");
            document.getElementById("instruction").classList.add("hidden-header");
            document.getElementById("finishButton").classList.add("hidden-button");
            document.getElementById("restartButton").classList.remove("hidden-button");
        });

        const restartButton = document.getElementById("restartButton");
        restartButton.addEventListener("click", () => {
            location.reload();
        });

        canvas.addEventListener("mousedown", (e) => {
            if (drawingEnabled) {
                isDrawing = true;
                if (isErasing) {
                    erase(e.offsetX, e.offsetY);
                } else {
                    draw(e.offsetX, e.offsetY);
                }
            }
        });

        canvas.addEventListener("mousemove", (e) => {
            if (drawingEnabled && isDrawing) {
                if (isErasing) {
                    erase(e.offsetX, e.offsetY);
                } else {
                    draw(e.offsetX, e.offsetY);
                }
            }
        });

        canvas.addEventListener("mouseup", () => {
            isDrawing = false;
            ctx.beginPath();
        });

        function draw(x, y) {
            ctx.lineWidth = brushSize;
            ctx.lineCap = "round";
            ctx.strokeStyle = currentColor;

            ctx.lineTo(x, y);
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(x, y);
        }

        function erase(x, y) {
            ctx.lineWidth = brushSize * 2;
            ctx.lineCap = "round";
            ctx.strokeStyle = "black";

            ctx.lineTo(x, y);
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(x, y);
        }

        function fillCanvas() {
            ctx.fillStyle = currentColor;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        const words = ["Apfel", "Haus", "Sonne", "Baum", "Auto", "Blume", "Katze", "Hund", "Ball", "Buch"];

        function displayRandomWords() {
            const wordListDiv = document.getElementById("wordList");
            wordListDiv.innerHTML = "";

            const randomWords = [];
            const usedIndices = [];

            while (randomWords.length < 3) {
                const randomIndex = Math.floor(Math.random() * words.length);
                if (!usedIndices.includes(randomIndex)) {
                    randomWords.push(words[randomIndex]);
                    usedIndices.push(randomIndex);
                }
            }

            randomWords.forEach(word => {
                const wordSpan = document.createElement("span");
                wordSpan.textContent = word;
                wordSpan.addEventListener("click", () => {
                    displaySelectedWord(word);
                });
                wordListDiv.appendChild(wordSpan);
            });
        }

        displayRandomWords();

        const selectedColorDiv = document.getElementById("selectedColor");

        function updateSelectedColor() {
            selectedColorDiv.style.backgroundColor = currentColor;
        }

        updateSelectedColor();

        function displaySelectedWord(word) {
            const selectedWordDiv = document.getElementById("selectedWord");
            selectedWordDiv.textContent = word;
            document.getElementById("wordList").style.display = "none";
        }
    </script>
</body>
</html>
