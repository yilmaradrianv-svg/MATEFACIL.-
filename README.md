<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MateFácil - Calculadora con Explicación Detallada</title>
    <style>
        :root {
            --bg-color: #121212;
            --calc-bg: #1e1e1e;
            --primary: #ff9f0a;
            --secondary: #a6a6a6;
            --btn-dark: #2d2d2d;
            --text-color: #ffffff;
        }
        body {
            font-family: 'Segoe UI', sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 10px;
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .app-container {
            width: 100%;
            max-width: 420px;
            background: var(--calc-bg);
            padding: 20px;
            border-radius: 25px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            box-sizing: border-box;
        }
        h1 { text-align: center; color: var(--primary); margin: 0 0 5px 0; font-size: 1.5rem; }
        p.subtitle { text-align: center; color: #888; margin: 0 0 10px 0; font-size: 0.85rem; }
        
        .categories {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 6px;
            margin-bottom: 15px;
        }
        .cat-btn {
            background-color: #2d2d2d;
            border: 1px solid #444;
            padding: 8px;
            border-radius: 8px;
            font-size: 0.85rem;
            font-weight: bold;
            color: #ccc;
            cursor: pointer;
            text-align: center;
            transition: all 0.2s;
        }
        .cat-btn.active {
            background-color: var(--primary);
            color: #000;
            border-color: var(--primary);
        }

        .display-screen {
            background-color: #000;
            border-radius: 15px;
            padding: 15px;
            margin-bottom: 15px;
            text-align: right;
            min-height: 70px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            box-sizing: border-box;
        }
        .expression-history {
            color: #777;
            font-size: 0.9rem;
            overflow-x: auto;
        }
        .main-result {
            color: #fff;
            font-size: 2rem;
            font-weight: bold;
            overflow-x: auto;
        }

        .keypad {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
        }
        .key {
            background-color: var(--btn-dark);
            border: none;
            border-radius: 50%;
            width: 100%;
            aspect-ratio: 1;
            font-size: 1.2rem;
            font-weight: bold;
            color: #fff;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background 0.2s, transform 0.1s;
        }
        .key:active { transform: scale(0.92); }
        .key.gray { background-color: var(--secondary); color: #000; }
        .key.orange { background-color: var(--primary); color: #000; }
        .key.wide {
            grid-column: span 2;
            border-radius: 35px;
            aspect-ratio: auto;
            height: 55px;
        }

        #explicacion-box {
            margin-top: 15px;
            background: #252525;
            border-radius: 15px;
            padding: 15px;
            display: none;
            font-size: 0.9rem;
            line-height: 1.5;
            color: #ddd;
            border-left: 5px solid var(--primary);
            max-height: 300px;
            overflow-y: auto;
        }
        #explicacion-box h3 { color: var(--primary); margin-top: 0; font-size: 1.1rem; }
        .step-card {
            background: #2f2f2f;
            padding: 10px;
            border-radius: 8px;
            margin-bottom: 8px;
            border-left: 3px solid var(--primary);
        }
    </style>
</head>
<body>

<div class="app-container">
    <h1>MateFácil</h1>
    <p class="subtitle">Calculadora con Explicación Detallada Paso a Paso</p>

    <div class="categories">
        <div class="cat-btn active" id="btn-basica" onclick="cambiarModo('basica')">🔢 Operaciones</div>
        <div class="cat-btn" id="btn-porcentaje" onclick="cambiarModo('porcentaje')">📈 Porcentajes</div>
        <div class="cat-btn" id="btn-potencia" onclick="cambiarModo('potencia')">✖️ Potencias</div>
        <div class="cat-btn" id="btn-raiz" onclick="cambiarModo('raiz')">➗ Raíces</div>
    </div>

    <div class="display-screen">
        <div id="history" class="expression-history">0</div>
        <div id="screen" class="main-result">0</div>
    </div>

    <div class="keypad">
        <button class="key gray" onclick="limpiar()">C</button>
        <button class="key gray" onclick="agregar('%')">%</button>
        <button class="key gray" onclick="agregar('/')">÷</button>
        <button class="key orange" onclick="agregar('*')">×</button>

        <button class="key" onclick="agregar('7')">7</button>
        <button class="key" onclick="agregar('8')">8</button>
        <button class="key" onclick="agregar('9')">9</button>
        <button class="key orange" onclick="agregar('-')">-</button>

        <button class="key" onclick="agregar('4')">4</button>
        <button class="key" onclick="agregar('5')">5</button>
        <button class="key" onclick="agregar('6')">6</button>
        <button class="key orange" onclick="agregar('+')">+</button>

        <button class="key" onclick="agregar('1')">1</button>
        <button class="key" onclick="agregar('2')">2</button>
        <button class="key" onclick="agregar('3')">3</button>
        <button class="key gray" onclick="borrarUltimo()">⌫</button>

        <button class="key" onclick="agregar('0')">0</button>
        <button class="key" onclick="agregar('.')">.</button>
        <button class="key wide orange" onclick="calcularYExplicar()">= Explicar</button>
    </div>

    <div id="explicacion-box"></div>
</div>

<script>
    let expresion = "";
    let modoActual = "basica";

    function cambiarModo(modo) {
        modoActual = modo;
        document.querySelectorAll('.cat-btn').forEach(b => b.classList.remove('active'));
        document.getElementById('btn-' + modo).classList.add('active');
        limpiar();
        
        let screen = document.getElementById('screen');
        if(modo === 'porcentaje') screen.innerText = "Ej: 20";
        if(modo === 'potencia') screen.innerText = "Ej: 2,3 (base,exp)";
        if(modo === 'raiz') screen.innerText = "Ej: 49";
    }

    function actualizarPantalla() {
        document.getElementById('screen').innerText = expresion === "" ? "0" : expresion;
    }

    function agregar(val) {
        expresion += val;
        actualizarPantalla();
    }

    function limpiar() {
        expresion = "";
        actualizarPantalla();
        document.getElementById('history').innerText = "0";
        document.getElementById('explicacion-box').style.display = "none";
    }

    function borrarUltimo() {
        expresion = expresion.slice(0, -1);
        actualizarPantalla();
    }

    function calcularYExplicar() {
        if (!expresion) return;
        
        document.getElementById('history').innerText = expresion;
        let div = document.getElementById('explicacion-box');
        div.style.display = "block";
        let html = `<h3>📖 Desglose Numérico Paso a Paso</h3>`;

        try {
            let exprJS = expresion.replace(/×/g, '*').replace(/÷/g, '/');

            if (modoActual === 'basica') {
                if (exprJS.includes('+')) {
                    let partes = exprJS.split('+');
                    let a = parseFloat(partes[0]), b = parseFloat(partes[1]);
                    let res = a + b;
                    let uA = a % 10, uB = b % 10;
                    let dA = Math.floor(a / 10) * 10, dB = Math.floor(b / 10) * 10;
                    html += `
                        <div class="step-card"><b>Paso 1: Alinear cantidades</b><br>Tenemos los números <b>${a}</b> y <b>${b}</b>.</div>
                        <div class="step-card"><b>Paso 2: Sumar unidades</b><br>Sumamos las unidades: ${uA} + ${uB} = <b>${uA + uB}</b>.</div>
                        <div class="step-card"><b>Paso 3: Sumar decenas</b><br>Sumamos las decenas: ${dA} + ${dB} = <b>${dA + dB}</b>.</div>
                        <div class="step-card" style="border-left-color: var(--primary);"><b>✅ Resultado final:</b> ${res}</div>
                    `;
                } else if (exprJS.includes('-')) {
                    let partes = exprJS.split('-');
                    let a = parseFloat(partes[0]), b = parseFloat(partes[1]);
                    let res = a - b;
                    let uA = a % 10, uB = b % 10;
                    let dA = Math.floor(a / 10) * 10, dB = Math.floor(b / 10) * 10;
                    html += `
                        <div class="step-card"><b>Paso 1: Identificar cantidades</b><br>Minuendo: <b>${a}</b>, Sustraendo: <b>${b}</b>.</div>
                        <div class="step-card"><b>Paso 2: Restar unidades</b><br>Restamos unidades: ${uA} - ${uB}.</div>
                        <div class="step-card"><b>Paso 3: Restar decenas</b><br>Restamos las decenas: ${dA} - ${dB}.</div>
                        <div class="step-card" style="border-left-color: var(--primary);"><b>✅ Resultado final:</b> ${res}</div>
                    `;
                } else if (exprJS.includes('*')) {
                    let partes = exprJS.split('*');
                    let a = parseFloat(partes[0]), b = parseFloat(partes[1]);
                    let res = a * b;
                    html += `
                        <div class="step-card"><b>Paso 1: Plantear factores</b><br>Multiplicamos <b>${a}</b> por <b>${b}</b> de forma directa.</div>
                        <div class="step-card" style="border-left-color: var(--primary);"><b>✅ Resultado final:</b> ${res}</div>
                    `;
                } else if (exprJS.includes('/')) {
                    let partes = exprJS.split('/');
                    let a = parseFloat(partes[0]), b = parseFloat(partes[1]);
                    if(b === 0) { alert("No se puede dividir entre cero"); return; }
                    let res = a / b;
                    html += `
                        <div class="step-card"><b>Paso 1: Plantear la división</b><br>Dividendo: <b>${a}</b>, Divisor: <b>${b}</b>.</div>
                        <div class="step-card"><b>Paso 2: Calcular cociente exacto</b><br>Buscamos cuántas veces cabe ${b} en ${a}.</div>
                        <div class="step-card" style="border-left-color: var(--primary);"><b>✅ Resultado final:</b> ${res}</div>
                    `;
                } else {
                    let res = eval(exprJS);
                    html += `<div class="step-card"><b>✅ Resultado:</b> ${res}</div>`;
                }
            } 
            else if (modoActual === 'porcentaje') {
                let p = parseFloat(expresion);
                let n = parseFloat(prompt("¿De qué número total quieres calcular el " + p + "%?", "150"));
                if (isNaN(n)) return;
                let decimal = p / 100;
                let res = n * decimal;
                html += `
                    <div class="step-card"><b>Paso 1: Convertir porcentaje a decimal</b><br>${p} ÷ 100 = <b>${decimal}</b></div>
                    <div class="step-card"><b>Paso 2: Multiplicar por el total</b><br>${n} × ${decimal} = <b>${res}</b></div>
                    <div class="step-card" style="border-left-color: var(--primary);"><b>✅ El ${p}% de ${n} es:</b> ${res}</div>
                `;
            } 
            else if (modoActual === 'potencia') {
                let nums = expresion.split(',');
                let base = parseFloat(nums[0]);
                let exponente = parseFloat(nums[1]) || 2;
                let res = Math.pow(base, exponente);
                html += `
                    <div class="step-card"><b>Paso 1: Analizar base y exponente</b><br>Base: <b>${base}</b>, Exponente: <b>${exponente}</b></div>
                    <div class="step-card"><b>Paso 2: Desarrollar multiplicación</b><br>Desglose: ${Array(exponente).fill(base).join(' × ')} = <b>${res}</b></div>
                    <div class="step-card" style="border-left-color: var(--primary);"><b>✅ Resultado final:</b> ${res}</div>
                `;
            } 
            else if (modoActual === 'raiz') {
                let num = parseFloat(expresion);
                let res = Math.sqrt(num);
                html += `
                    <div class="step-card"><b>Paso 1: Plantear raíz cuadrada</b><br>Buscamos un número que multiplicado por sí mismo dé <b>${num}</b>.</div>
                    <div class="step-card"><b>Paso 2: Comprobación</b><br>${res} × ${res} = ${num}</div>
                    <div class="step-card" style="border-left-color: var(--primary);"><b>✅ Resultado final:</b> ${res}</div>
                `;
            }

            let resultadoFinal = eval(exprJS);
            document.getElementById('screen').innerText = resultadoFinal;
            div.innerHTML = html;

        } catch (e) {
            div.innerHTML = `<p style="color: #ff6b6b;">⚠️ Operación inválida. Revisa los datos ingresados.</p>`;
        }
    }
</script>

</body>
</html>
