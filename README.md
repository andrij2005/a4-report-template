<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Звіт A4 - Інтерактивний Шаблон з KaTeX</title>
    
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css" integrity="sha384-GvrS47g8e2J4J+Y/0vNnF8X08n/J/R7iSg6Z7v0E9X1P2p08B1V3N5F8t4D01Yl" crossorigin="anonymous">
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js" integrity="sha384-k7HkFv+g077F5rG5Wv3b1Z1S1nF5F4B1T7wF2P1Y2p08B1V3N5F8t4D01Yl" crossorigin="anonymous"></script>
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/contrib/auto-render.min.js" integrity="sha384-M+p5t6e/e5JvRz2a3F0Q1p2E5N7G5K5z1P1p08B1V3N5F8t4D01Yl" crossorigin="anonymous" onload="rerenderAll()"></script>
    
    <style>
        /* Стилі для емуляції сторінки A4 */
        body {
            background: rgb(204, 204, 204);
            margin: 0;
            padding: 0;
            font-family: 'Times New Roman', Times, serif;
            font-size: 12pt;
        }

        .page {
            background: white;
            margin: 1cm auto;
            width: 21cm;
            min-height: 29.7cm;
            padding: 2cm;
            box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
            box-sizing: border-box;
            page-break-after: always;
            word-wrap: break-word; 
        }
        
        .page:last-child {
            page-break-after: avoid;
        }

        /* Стиль для спеціального маркування тексту (не впливає на звичайне виділення) */
        .formula-candidate {
            background-color: #ffeb3b80; /* Жовтий фон для позначення */
            color: #d81b60; /* Яскравий колір тексту */
            padding: 1px 3px;
            border-radius: 3px;
            cursor: pointer;
            user-select: none; /* Забороняємо виділяти сам span */
        }
        
        .page > div {
             /* Стиль для contenteditable поля */
            min-height: 25cm;
            outline: none;
            white-space: pre-wrap; /* Зберігає форматування при вставленні (Enter) */
            caret-color: black;
        }

        /* Стилі для панелі управління */
        #controls {
            position: fixed;
            top: 10px;
            right: 10px;
            padding: 10px;
            background: #f8f8f8;
            border: 1px solid #ccc;
            border-radius: 5px;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            gap: 5px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        }
        
        #controls hr {
            width: 80%;
            border: 0;
            border-top: 1px solid #ccc;
        }

        /* Стилі для друку */
        @media print {
            #controls {
                display: none; 
            }
            body, .page {
                background: white;
                margin: 0;
                box-shadow: none;
            }
            .page {
                padding: 2cm;
                width: 100%;
                min-height: auto;
            }
        }
    </style>
    
    <script>
        document.addEventListener("DOMContentLoaded", function() {
            addPage(); // Ініціалізація першої сторінки
        });

        const RENDER_CONFIG = {
            // Конфігурація для авто-рендерингу всього документа
            delimiters: [
                {left: '$$', right: '$$', display: true},
                {left: '$', right: '$', display: false},
                {left: '\\[', right: '\\]', display: true},
                {left: '\\(', right: '\\)', display: false}
            ],
            throwOnError : false
        };

        // Функція для повного перетворення всього документа (Перетворити ВСЕ)
        function rerenderAll() {
            const container = document.getElementById("content-container");
            renderMathInElement(container, RENDER_CONFIG);
        }
        
        // Функція для додавання нової сторінки
        function addPage() {
            const container = document.getElementById('content-container');
            const newPage = document.createElement('div');
            newPage.className = 'page';
            
            newPage.innerHTML = `
                <div contenteditable="true" spellcheck="false">
                    Вставте тут текст з формулами LaTeX. Наприклад, рівняння Ейлера: $e^{i\pi} + 1 = 0$.
                </div>
            `;
            container.appendChild(newPage);
            
            // Якщо сторінка не перша, додаємо візуальний роздільник для кращої навігації
            if (container.children.length > 1) {
                const breakDiv = document.createElement('div');
                breakDiv.className = 'page-break-divider'; // Використовуємо окремий клас
                container.insertBefore(breakDiv, newPage);
            }
            
            // Фокусуємо курсор на новій сторінці
            newPage.querySelector('[contenteditable]').focus();
            rerenderAll();
        }

        // Функція для дублювання Enter (вставлення розриву рядка)
        function insertBreak() {
            document.execCommand('insertHTML', false, '<br>');
        }

        // Функція для маркування (виділення) або зняття маркування
        function markSelection(mark) {
            const className = 'formula-candidate';
            const selection = window.getSelection();
            if (selection.rangeCount === 0) return;

            if (mark) {
                // Маркування: обгортаємо виділення у span з класом
                document.execCommand('insertHTML', false, `<span class="${className}">` + selection.toString() + '</span>');
            } else {
                // Зняття маркування: видаляємо span (зберігаючи вміст)
                const range = selection.getRangeAt(0);
                const parentElement = range.commonAncestorContainer.parentNode;

                if (parentElement.classList && parentElement.classList.contains(className)) {
                    // Видаляємо span, зберігаючи його вміст
                    const content = parentElement.innerHTML;
                    // OuterHTML замінює сам елемент і його вміст на лише вміст
                    parentElement.outerHTML = content; 
                }
            }
        }

        // Посилена функція для перетворення маркованого тексту у формули KaTeX
        function convertFormulas() {
            const candidates = document.querySelectorAll('.formula-candidate');
            
            candidates.forEach(span => {
                let formulaText = span.textContent; 
                
                // --- Агресивне Очищення та Виправлення Синтаксису ---
                
                // 1. Видалення фігурних дужок, які часто з'являються при копіюванні { та }
                formulaText = formulaText.replace(/\{/g, '').replace(/\}/g, ''); 
                
                // 2. Видалення зайвих пробілів навколо $$, $ та \
                formulaText = formulaText.replace(/\s*\$\$\s*/g, '$$'); 
                formulaText = formulaText.replace(/\s*\$\s*/g, '$');
                formulaText = formulaText.replace(/\\ /g, '\\'); // Прибираємо пробіли після слешів
                
                // 3. Обрізання пробілів тільки на початку і в кінці всього рядка
                formulaText = formulaText.trim();
                
                // --- Запускаємо рендеринг KaTeX ---
                let targetElement = span;
                targetElement.innerHTML = ''; // Очищаємо вміст для рендерингу
                
                try {
                    const isDisplayMode = formulaText.startsWith('$$') && formulaText.endsWith('$$');
                    
                    let renderText = formulaText;
                    if (isDisplayMode) {
                         // Обрізаємо $$ з початку і кінця для коректної передачі в KaTeX
                         renderText = renderText.substring(2, renderText.length - 2).trim();
                    }
                    
                    katex.render(renderText, targetElement, {
                        displayMode: isDisplayMode,
                        throwOnError: false 
                    });
                    
                    // Якщо рендеринг успішний, видаляємо клас маркування
                    span.classList.remove('formula-candidate');
                    span.style.color = 'inherit'; 
                    span.style.background = 'inherit'; 
                    
                } catch (e) {
                    // Якщо рендеринг не вдався, позначаємо червоним
                    console.error("Помилка рендерингу KaTeX:", e);
                    span.style.color = 'red'; 
                    span.style.background = '#f4433650'; 
                }
            });
        }
    </script>
</head>
<body>
    <div id="controls">
        <button onclick="addPage()">➕ Новий Аркуш</button>
        <button onclick="insertBreak()">↩️ Enter</button>
        <hr>
        <button onclick="markSelection(true)">✏️ Виділити Текст</button>
        <button onclick="markSelection(false)">🚫 Відмінити Виділення</button>
        <button onclick="convertFormulas()">⚛️ Перетворити у Формулу</button>
        <hr>
        <button onclick="rerenderAll()">🔄 Перетворити ВСЕ (Авто)</button>
    </div>
    
    <div id="content-container">
        </div>
</body>
</html>
