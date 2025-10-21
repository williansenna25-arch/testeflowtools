import express from 'express';
import fs from 'fs';
import path from 'path';
import bodyParser from 'body-parser';
import cors from 'cors';

const app = express();
app.use(cors());
app.use(bodyParser.json());

const DB_FILE = './data/db.json';

// rota gerar presell
app.post('/api/gerar-presell', (req, res) => {
  const { modelo, link_afiliado } = req.body;
  const templatePath = path.join('./templates', `${modelo}.html`);
  let html = fs.readFileSync(templatePath, 'utf8');
  html = html.replaceAll('{{link_afiliado}}', link_afiliado);
  res.json({ html });
});

// rota registrar cliques
app.post('/api/click', (req, res) => {
  const db = JSON.parse(fs.readFileSync(DB_FILE));
  db.cliques.push({
    campanha: req.body.campanha,
    ip: req.ip,
    device: req.headers['user-agent'],
    hora: new Date().toISOString()
  });
  fs.writeFileSync(DB_FILE, JSON.stringify(db, null, 2));
  res.json({ success: true });
});

// webhook de conversões
app.post('/api/webhook', (req, res) => {
  const db = JSON.parse(fs.readFileSync(DB_FILE));
  db.conversoes.push(req.body);
  fs.writeFileSync(DB_FILE, JSON.stringify(db, null, 2));
  res.sendStatus(200);
});

app.listen(3000, () => console.log("Servidor rodando em http://localhost:3000"));
