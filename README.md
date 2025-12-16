<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>David Barber - Reservas</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background: linear-gradient(to right, #2c3e50, #4ca1af);
      color: #fff;
      text-align: center;
    }

    header {
      background: #111;
      padding: 20px;
    }

    header h1 {
      margin: 0;
      font-size: 2rem;
      letter-spacing: 2px;
    }

    .container {
      margin: 40px auto;
      padding: 20px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 15px;
      width: 90%;
      max-width: 400px;
      box-shadow: 0 0 15px rgba(0,0,0,0.5);
    }

    label {
      display: block;
      margin: 15px 0 5px;
      text-align: left;
    }

    input, select, button {
      text-align: center;
      width: 100%;
      padding: 10px;
      border: none;
      border-radius: 8px;
      margin-bottom: 15px;
      font-size: 1rem;
    }

    input, select {
      color: #333;
    }

    button {
      background: #e67e22;
      color: white;
      font-weight: bold;
      cursor: pointer;
      transition: 0.3s;
    }

    button:hover {
      background: #d35400;
    }

    .mensagem {
      margin-top: 20px;
      font-size: 1.1rem;
      font-weight: bold;
    }

    .lista-reservas {
      margin-top: 30px;
      text-align: left;
    }

    .lista-reservas h2 {
      font-size: 1.3rem;
      border-bottom: 1px solid #fff;
      padding-bottom: 5px;
    }

    .reserva {
      background: rgba(255, 255, 255, 0.15);
      margin: 10px 0;
      padding: 10px;
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <header>
    <h1>💈 David Barber 💈</h1>
    <p>Reserve seu horário online!</p>
  </header>

  <div class="container">
    <form id="formReserva">
      <label for="nome">Seu nome:</label>
      <input type="text" id="nome" required>

      <label for="data">Escolha a data:</label>
      <input type="date" id="data" required>

      <label for="hora">Escolha o horário:</label>
      <select id="hora" required>
        <option value="">Selecione</option>
      </select>

      <button type="submit">Confirmar Reserva</button>
    </form>

    <div class="mensagem" id="mensagem"></div>

    <div class="lista-reservas">
      <h2>📅 Reservas do dia selecionado</h2>
      <div id="lista"></div>
    </div>
  </div>

  <!-- ====================== FIREBASE ====================== -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-app.js";
    import { getFirestore, collection, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-firestore.js";
  
    const firebaseConfig = {
      apiKey: "AIzaSyDCju1ONpNxsnx6TTrL6yWNfJDq4uiwThs",
      authDomain: "marcarhorario-a732c.firebaseapp.com",
      projectId: "marcarhorario-a732c",
      storageBucket: "marcarhorario-a732c.appspot.com",
      messagingSenderId: "710229850617",
      appId: "1:710229850617:web:c270f4b1eb2cc3e322a528",
      measurementId: "G-ZFYVE54ER3"
    };
  
    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    const form = document.getElementById("formReserva");
    const mensagem = document.getElementById("mensagem");
    const lista = document.getElementById("lista");
    const inputData = document.getElementById("data");
    const selectHora = document.getElementById("hora");

    const horarios = [
      "09:00", "09:30", "10:00", "10:30", "11:00",
      "14:00", "14:30", "15:00", "15:30", "16:00",
      "16:30", "17:00", "17:30", "18:00", "18:30",
      "19:30", "20:00"
    ];

    // Limitar datas em até 1 mês
    const hoje = new Date();
    const maxDate = new Date();
    maxDate.setMonth(maxDate.getMonth() + 1);
    const formatar = (d) => d.toISOString().split("T")[0];
    inputData.min = formatar(hoje);
    inputData.max = formatar(maxDate);

    // Atualiza horários e lista quando a data muda
    inputData.addEventListener("change", async () => {
      const dataSelecionada = new Date(inputData.value);
      if (dataSelecionada.getDay() === 0) {
        alert("⚠️ Não é possível marcar aos domingos.");
        inputData.value = "";
        selectHora.innerHTML = '<option value="">Selecione</option>';
        lista.innerHTML = "";
      } else {
        await atualizarHorarios(inputData.value);
        await atualizarLista(inputData.value);
      }
    });

    // Preencher horários disponíveis para a data selecionada
    async function atualizarHorarios(dataSelecionada) {
      selectHora.innerHTML = '<option value="">Selecione</option>';

      const reservasSnapshot = await getDocs(collection(db, "agendamentos"));
      const reservas = reservasSnapshot.docs.map(doc => doc.data());

      horarios.forEach((hora) => {
        const option = document.createElement("option");
        option.value = hora;
        option.textContent = hora;

        const reservado = reservas.find(r => r.data === dataSelecionada && r.hora === hora);
        if (reservado) {
          option.disabled = true;
          option.textContent += " (Indisponível)";
        }

        selectHora.appendChild(option);
      });
    }

    // Mostrar somente reservas do dia selecionado
    async function atualizarLista(dataSelecionada) {
      lista.innerHTML = "";

      if (!dataSelecionada) return;

      const reservasSnapshot = await getDocs(collection(db, "agendamentos"));
      const reservasDoDia = reservasSnapshot.docs
        .map(doc => doc.data())
        .filter(r => r.data === dataSelecionada);

      if (reservasDoDia.length === 0) {
        lista.innerHTML = "<p>Nenhuma reserva feita ainda.</p>";
        return;
      }

      reservasDoDia.forEach(r => {
        const div = document.createElement("div");
        div.classList.add("reserva");
        div.textContent = `✅ ${r.nome} - ${r.hora}`;
        lista.appendChild(div);
      });
    }

    // Captura envio do formulário
    form.addEventListener("submit", async (e) => {
      e.preventDefault();

      const nome = document.getElementById("nome").value;
      const data = document.getElementById("data").value;
      const hora = document.getElementById("hora").value;

      if (nome && data && hora) {
        try {
          await addDoc(collection(db, "agendamentos"), { nome, data, hora, criadoEm: new Date() });

          mensagem.style.color = "lightgreen";
          mensagem.textContent = `✅ Reserva confirmada para ${nome} em ${data} às ${hora}.`;

          form.reset();
          await atualizarHorarios(data);
          await atualizarLista(data);

        } catch (error) {
          console.error("Erro ao salvar:", error);
          mensagem.style.color = "red";
          mensagem.textContent = "❌ Erro ao salvar: " + error.message;
        }
      } else {
        mensagem.style.color = "yellow";
        mensagem.textContent = "⚠️ Preencha todos os campos!";
      }
    });
  </script>
</body>
</html>
