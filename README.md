<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Звіт A4 - Інтерактивний Шаблон</title>
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
            word-wrap: break-word; /* Для коректного перенесення довгих рядків */
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
        
        /* Стилі для друку */
        @media print {
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
            #controls {
                display: none; /* Приховуємо кнопки при друку */
            }
        }
    </style>
    
    <script>
        document.addEventListener("DOMContentLoaded", function() {
            // Ініціалізація першої сторінки та кнопок
            addPage();
        });

        const RENDER_CONFIG = {
            // Використовуємо $...$ для inline-формул і $$...$$ для display-формул
            delimiters: [
                {left: '$$', right: '$$', display: true},
                {left: '$', right: '$', display: false},
                {left: '\\[', right: '\\]', display: true},
                {left: '\\(', right: '\\)', display: false}
            ],
            throwOnError : false
        };

        // Функція для повного перетворення всього документа (аналог "Перетворити ВСЕ")
        function rerenderAll() {
            const container = document.getElementById("content-container");
            renderMathInElement(container, RENDER_CONFIG);
        }
        
        // Функція для додавання нової сторінки
        function addPage() {
            const container = document.getElementById('content-container');
            const newPage = document.createElement('div');
            newPage.className = 'page';
            // Вміст сторінки можна редагувати
            newPage.innerHTML = `
                <div contenteditable="true" spellcheck="false" style="min-height: 25cm; outline: none; white-space: pre-wrap; caret-color: black;">
                    Вставте тут текст з формулами LaTeX.
                    Наприклад, рівняння маси: $E=mc^2$.
                </div>
            `;
            container.appendChild(newPage);
            // Викликаємо загальний рендеринг, щоб KaTeX опрацював новий вміст, якщо він є
            rerenderAll();
        }

        // Функція для дублювання Enter (вставлення розриву рядка)
        function insertBreak() {
            // document.execCommand працює добре для contenteditable
            document.execCommand('insertHTML', false, '<br>');
        }

        // Функція для маркування (виділення) або зняття маркування
        function markSelection(mark) {
            const className = 'formula-candidate';
            const selection = window.getSelection();
            if (selection.rangeCount === 0) return;

            if (mark) {
                // 1. Маркування: обгортаємо виділення у span з класом
                document.execCommand('insertHTML', false, `<span class="${className}">` + selection.toString() + '</span>');
            } else {
                // 2. Зняття маркування: видаляємо клас/тег з виділеної ділянки
                const range = selection.getRangeAt(0);
                const parentElement = range.commonAncestorContainer.parentNode;

                // Перевіряємо, чи батьківський елемент є нашим маркованим span
                if (parentElement.classList && parentElement.classList.contains(className)) {
                    // Видаляємо span, зберігаючи його вміст
                    const content = parentElement.innerHTML;
                    parentElement.outerHTML = content; 
                }
            }
        }

        // Функція для перетворення маркованого тексту у формули KaTeX
        function convertFormulas() {
            const candidates = document.querySelectorAll('.formula-candidate');
            
            candidates.forEach(span => {
                let formulaText = span.textContent.trim();
                
                // --- Очищення та виправлення синтаксису (для коректного копіювання) ---
                // Видалення фігурних дужок, які можуть з'явитися при копіюванні
                formulaText = formulaText.replace(/\{/g, '').replace(/\}/g, '').trim();
                
                // Видалення зайвих пробілів навколо $$, якщо вони є
                formulaText = formulaText.replace(/\s*\$\$\s*/g, '$$'); 

                // --- Запускаємо рендеринг KaTeX ---
                let targetElement = span;
                targetElement.innerHTML = '';
                
                try {
                    katex.render(formulaText, targetElement, {
                        displayMode: formulaText.startsWith('$$') && formulaText.endsWith('$$'), // Блочний режим, якщо є $$
                        throwOnError: false // Не викидати помилку
                    });
                    
                    // Якщо рендеринг успішний, видаляємо клас маркування
                    span.classList.remove('formula-candidate');
                    span.style.color = 'inherit'; // Повертаємо нормальний колір
                    span.style.background = 'inherit'; // Повертаємо нормальний фон
                    
                } catch (e) {
                    // Якщо рендеринг не вдався, залишаємо маркування, але міняємо колір на червоний
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
        <button onclick="rerenderAll()">🔄 Перетворити ВСЕ</button>
    </div>
    
    <div id="content-container">
        </div>
</body>
</html>

