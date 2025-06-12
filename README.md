<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Juego de Memoria por Niveles</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f0f0f0;
      text-align: center;
      padding: 20px;
    }

    h1 {
      color: #2c3e50;
    }

    .board {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
      gap: 10px;
      max-width: 700px;
      margin: auto;
    }

    .card {
      background: #3498db;
      color: white;
      padding: 20px;
      height: 120px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      perspective: 1000px;
      border-radius: 8px;
      transition: transform 0.3s;
    }

    .card.flipped {
      background: #2ecc71;
    }

    .card.incorrect {
      animation: shake 0.3s;
    }

    @keyframes shake {
      0% { transform: translateX(0); }
      25% { transform: translateX(-5px); }
      50% { transform: translateX(5px); }
      75% { transform: translateX(-5px); }
      100% { transform: translateX(0); }
    }

    .info {
      margin: 15px;
    }

    .diploma {
      display: none;
      background: #fff;
      border: 2px solid #2ecc71;
      padding: 30px;
      max-width: 500px;
      margin: 20px auto;
      border-radius: 12px;
    }

    button {
      padding: 10px 20px;
      background: #e67e22;
      border: none;
      color: white;
      border-radius: 5px;
      cursor: pointer;
    }
  </style>
</head>
<body>

<h1>Juego de Memoria: Seguridad y Salud en el Trabajo</h1>

<div class="info">
  <p>⏱️ Tiempo: <span id="timer">00:00</span></p>
  <p>❌ Errores: <span id="errors">0</span></p>
</div>

<div class="board" id="board"></div>

<div class="diploma" id="diploma">
  <h2>🎓 ¡Felicidades!</h2>
  <p>⏱️ Tiempo total: <span id="finalTime"></span></p>
  <p>❌ Total de errores: <span id="finalErrors"></span></p>
  <button onclick="reiniciarJuego()">Jugar de nuevo</button>
</div>

<br />
<button onclick="reiniciarJuego()">🔄 Reiniciar Juego</button>

<script>
  const niveles = [
    [
      ["Ergonomía", "Ciencia que busca adaptar el trabajo al trabajador para mejorar la eficiencia y reducir el riesgo de lesiones."],
      ["EPP", "Equipos o dispositivos que protegen al trabajador de riesgos que puedan amenazar su salud o seguridad."],
      ["Riesgo", "La posibilidad de que ocurra un evento que cause daño o pérdidas en el entorno laboral."]
    ],
    [
      ["Incidente", "Suceso no deseado que pudo haber causado daño, pero no lo hizo."],
      ["Accidente", "Evento inesperado que causa una lesión, daño o pérdida en el trabajo."],
      ["Peligro", "Fuente, situación o acto con potencial de causar daño en términos de lesiones, enfermedades o pérdidas."],
      ["Pausa Activa", "Ejercicio breve durante la jornada laboral para relajar el cuerpo y reducir el estrés físico y mental."]
    ],
    [
      ["SG-SST", "Conjunto de normas y procedimientos para prevenir accidentes y enfermedades laborales."],
      ["COPASST", "Comité Paritario de Seguridad y Salud en el Trabajo, encargado de promover la salud y seguridad en la empresa."],
      ["Matriz de Riesgos", "Herramienta que identifica, evalúa y clasifica los riesgos laborales para definir controles."],
      ["Enfermedad Laboral", "Afección de salud causada por la exposición prolongada a factores de riesgo relacionados con el trabajo."],
      ["Acto Inseguro", "Comportamiento del trabajador que puede provocar un accidente o enfermedad en el lugar de trabajo."]
    ]
  ];

  let nivelActual = 0;
  let primera = null;
  let segunda = null;
  let bloqueado = false;
  let errores = 0;
  let timerStarted = false;
  let startTime;
  let interval;

  const timerEl = document.getElementById("timer");
  const errorsEl = document.getElementById("errors");
  const board = document.getElementById("board");
  const diploma = document.getElementById("diploma");
  const finalTime = document.getElementById("finalTime");
  const finalErrors = document.getElementById("finalErrors");

  function iniciarTemporizador() {
    startTime = new Date();
    interval = setInterval(() => {
      const now = new Date();
      const diff = Math.floor((now - startTime) / 1000);
      const min = String(Math.floor(diff / 60)).padStart(2, '0');
      const sec = String(diff % 60).padStart(2, '0');
      timerEl.textContent = `${min}:${sec}`;
    }, 1000);
  }

  function detenerTemporizador() {
    clearInterval(interval);
  }

  function reiniciarJuego() {
    errores = 0;
    erroresEl.textContent = 0;
    nivelActual = 0;
    timerEl.textContent = "00:00";
    timerStarted = false;
    detenerTemporizador();
    diploma.style.display = "none";
    cargarNivel();
  }

  function cargarNivel() {
    board.innerHTML = "";
    const pares = niveles[nivelActual];
    let tarjetas = [];
    pares.forEach((par, index) => {
      tarjetas.push({ id: index, texto: par[0], tipo: "termino" });
      tarjetas.push({ id: index, texto: par[1], tipo: "definicion" });
    });

    tarjetas = tarjetas.sort(() => Math.random() - 0.5);

    tarjetas.forEach(data => {
      const div = document.createElement("div");
      div.className = "card";
      div.textContent = "?";
      div.dataset.id = data.id;
      div.dataset.texto = data.texto;
      div.dataset.tipo = data.tipo;
      div.onclick = () => voltearCarta(div);
      board.appendChild(div);
    });
  }

  function voltearCarta(carta) {
    if (bloqueado || carta.classList.contains("flipped")) return;

    if (!timerStarted) {
      timerStarted = true;
      iniciarTemporizador();
    }

    carta.textContent = carta.dataset.texto;
    carta.classList.add("flipped");

    if (!primera) {
      primera = carta;
    } else {
      segunda = carta;
      bloqueado = true;

      setTimeout(() => {
        const match = primera.dataset.id === segunda.dataset.id &&
                      primera.dataset.tipo !== segunda.dataset.tipo;

        if (match) {
          primera = segunda = null;
          if ([...board.children].every(c => c.classList.contains("flipped"))) {
            setTimeout(() => avanzarNivel(), 1000);
          }
          bloqueado = false;
        } else {
          errores++;
          errorsEl.textContent = errores;
          primera.classList.add("incorrect");
          segunda.classList.add("incorrect");
          setTimeout(() => {
            primera.classList.remove("flipped", "incorrect");
            segunda.classList.remove("flipped", "incorrect");
            primera.textContent = "?";
            segunda.textContent = "?";
            primera = segunda = null;
            bloqueado = false;
          }, 500);
        }
      }, 4000); // Pausa para lectura
    }
  }

  function avanzarNivel() {
    nivelActual++;
    if (nivelActual < niveles.length) {
      cargarNivel();
    } else {
      finalizarJuego();
    }
  }

  function finalizarJuego() {
    detenerTemporizador();
    diploma.style.display = "block";
    finalTime.textContent = timerEl.textContent;
    finalErrors.textContent = errores;
    board.innerHTML = "";
  }

  // Inicia el primer nivel al cargar
  cargarNivel();
</script>

</body>
</html>
