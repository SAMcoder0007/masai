# masai
healthcare system
# PROJECT STRUCTURE
healthcare-system/
├── backend/       # Node.js + MongoDB API
├── frontend/      # React UI
└── docker-compose.yml
**BACKEND CODE**
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

// MongoDB Connection
mongoose.connect('mongodb://mongo:27017/healthcare', { 
  useNewUrlParser: true, 
  useUnifiedTopology: true 
});

// Patient Schema
const PatientSchema = new mongoose.Schema({
  patientId: { type: String, unique: true },
  name: String,
  age: Number,
  visits: [{
    date: Date,
    diagnosis: String,
    prescriptions: [String]
  }]
}, { timestamps: true });

const Patient = mongoose.model('Patient', PatientSchema);

// Express App
const app = express();
app.use(cors());
app.use(express.json());

// Routes
app.post('/api/patients', async (req, res) => {
  const patient = new Patient(req.body);
  await patient.save();
  res.status(201).send(patient);
});

app.get('/api/patients', async (req, res) => {
  const patients = await Patient.find();
  res.send(patients);
});

// Start Server
app.listen(5000, () => {
  console.log('Backend running on port 5000');
});
**FRONT END CODE**
import React, { useState, useEffect } from 'react';

function App() {
  const [patients, setPatients] = useState([]);
  const [formData, setFormData] = useState({
    patientId: '',
    name: '',
    age: ''
  });

  // Fetch patients
  useEffect(() => {
    fetch('/api/patients')
      .then(res => res.json())
      .then(data => setPatients(data));
  }, []);

  // Add new patient
  const handleSubmit = (e) => {
    e.preventDefault();
    fetch('/api/patients', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    }).then(() => window.location.reload());
  };

  return (
    <div>
      <h1>Patient Management</h1>
      
      {/* Add Patient Form */}
      <form onSubmit={handleSubmit}>
        <input 
          placeholder="Patient ID"
          value={formData.patientId}
          onChange={(e) => setFormData({...formData, patientId: e.target.value})}
        />
        <button type="submit">Add Patient</button>
      </form>

      {/* Patient List */}
      <ul>
        {patients.map(p => (
          <li key={p._id}>{p.name} (ID: {p.patientId})</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
**DOCKER**


services:
  mongo:
    image: mongo:6.0
    ports: ["27017:27017"]
    volumes: [mongo-data:/data/db]

  backend:
    build: ./backend
    ports: ["5000:5000"]
    depends_on: [mongo]
    environment:
      MONGO_URI: "mongodb://mongo:27017/healthcare"

  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    depends_on: [backend]
  **BACKEND DOCKER**
  FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
**FRONTEND DOCKER**
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
