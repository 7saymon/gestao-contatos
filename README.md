// app.js - Arquivo principal do servidor

const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const contactRoutes = require("./routes/contactRoutes");

const app = express();
const PORT = process.env.PORT || 5000;

// Conectar ao MongoDB
mongoose
  .connect("mongodb://localhost:27017/gestao-contatos", {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => console.log("MongoDB conectado"))
  .catch((err) => console.log(err));

// Middlewares
app.use(cors());
app.use(express.json());
app.use("/api/contatos", contactRoutes);

app.listen(PORT, () => console.log(`Servidor rodando na porta ${PORT}`));

// models/Contact.js - Modelo de Contato
const mongoose = require("mongoose");

const ContactSchema = new mongoose.Schema({
  nome: { type: String, required: true },
  email: { type: String, required: true },
  telefone: { type: String, required: true },
  criadoEm: { type: Date, default: Date.now },
});

module.exports = mongoose.model("Contact", ContactSchema);

// routes/contactRoutes.js - Rotas para Contatos
const express = require("express");
const router = express.Router();
const Contact = require("../models/Contact");

// Criar novo contato
router.post("/", async (req, res) => {
  try {
    const contato = new Contact(req.body);
    await contato.save();
    res.status(201).json(contato);
  } catch (error) {
    res.status(500).json({ error: "Erro ao criar contato" });
  }
});

// Listar todos os contatos
router.get("/", async (req, res) => {
  try {
    const contatos = await Contact.find();
    res.json(contatos);
  } catch (error) {
    res.status(500).json({ error: "Erro ao buscar contatos" });
  }
});

// Atualizar contato
router.put("/:id", async (req, res) => {
  try {
    const contato = await Contact.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.json(contato);
  } catch (error) {
    res.status(500).json({ error: "Erro ao atualizar contato" });
  }
});

// Deletar contato
router.delete("/:id", async (req, res) => {
  try {
    await Contact.findByIdAndDelete(req.params.id);
    res.json({ message: "Contato removido" });
  } catch (error) {
    res.status(500).json({ error: "Erro ao deletar contato" });
  }
});

module.exports = router;

// frontend/index.html - Interface Simples para Gerenciamento de Contatos
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão de Contatos</title>
    <script defer src="script.js"></script>
</head>
<body>
    <h1>Gestão de Contatos</h1>
    <form id="contactForm">
        <input type="text" id="nome" placeholder="Nome" required>
        <input type="email" id="email" placeholder="Email" required>
        <input type="text" id="telefone" placeholder="Telefone" required>
        <button type="submit">Salvar</button>
    </form>
    <ul id="contactList"></ul>
</body>
</html>

// frontend/script.js - Lógica para Interação com API
const API_URL = "http://localhost:5000/api/contatos";

document.getElementById("contactForm").addEventListener("submit", async (e) => {
    e.preventDefault();
    const nome = document.getElementById("nome").value;
    const email = document.getElementById("email").value;
    const telefone = document.getElementById("telefone").value;
    
    const response = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ nome, email, telefone })
    });
    if (response.ok) location.reload();
});

async function carregarContatos() {
    const response = await fetch(API_URL);
    const contatos = await response.json();
    document.getElementById("contactList").innerHTML = contatos.map(c => `<li>${c.nome} - ${c.email} - ${c.telefone}</li>`).join("");
}
carregarContatos(); gestao-contatos
