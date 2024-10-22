# Cloud Native Monitoring Application

Creating a Cloud Native Monitoring Application involves building a system that collects, processes, and visualizes metrics from various services running in a cloud environment. Below, I’ll outline the steps to create a simple monitoring application using the MERN stack (MongoDB, Express.js, React, Node.js), integrated with a monitoring tool like Prometheus and Grafana.

**Project Structure**
Here’s a suggested project structure:

``java
cloud-native-monitoring/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   └── .env
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   └── public/
├── docker-compose.yml
└── prometheus/
    ├── prometheus.yml
``

**Step 1: Set Up the Backend**

**A. Create the Backend Directory Create the backend directory and navigate to it:**

``bash
mkdir backend
cd backend
Initialize a new Node.js project:
``

``bash
npm init -y
Install necessary dependencies:
``

``bash
npm install express mongoose dotenv cors body-parser
Create the server.js file:
``

``javascript
// backend/server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(bodyParser.json());

// MongoDB Connection
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

// Metrics Schema
const metricSchema = new mongoose.Schema({
    name: String,
    value: Number,
    timestamp: { type: Date, default: Date.now }
});

const Metric = mongoose.model('Metric', metricSchema);

// Routes
app.get('/metrics', async (req, res) => {
    const metrics = await Metric.find();
    res.json(metrics);
});

app.post('/metrics', async (req, res) => {
    const newMetric = new Metric(req.body);
    await newMetric.save();
    res.status(201).json(newMetric);
});

// Start the server
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
``

**Create a .env file to store environment variables:**

``bash
MONGO_URI=mongodb://mongo:27017/monitoring
``

**B. Create the Dockerfile for Backend**

``dockerfile
FROM node:14

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
``

**Step 2: Set Up the Frontend**

**A. Create the Frontend Directory Create the frontend directory and navigate to it:**

``bash
mkdir ../frontend
cd ../frontend
Create a new React app:
``

``bash
npx create-react-app .
Modify the package.json to include a proxy for API calls:
``

``json
// frontend/package.json
{
  // ... other configurations
  "proxy": "http://backend:5000"
}
``

**Create a simple Monitoring Dashboard in src/App.js:**

``javascript
// frontend/src/App.js
import React, { useEffect, useState } from 'react';

function App() {
  const [metrics, setMetrics] = useState([]);
  const [metricName, setMetricName] = useState('');
  const [metricValue, setMetricValue] = useState('');

  useEffect(() => {
    fetch('/metrics')
      .then(response => response.json())
      .then(data => setMetrics(data));
  }, []);

  const addMetric = async () => {
    const response = await fetch('/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: metricName, value: parseFloat(metricValue) })
    });
    const newMetric = await response.json();
    setMetrics([...metrics, newMetric]);
    setMetricName('');
    setMetricValue('');
  };

  return (
    <div>
      <h1>Cloud Native Monitoring Dashboard</h1>
      <input
        type="text"
        value={metricName}
        onChange={e => setMetricName(e.target.value)}
        placeholder="Metric Name"
      />
      <input
        type="number"
        value={metricValue}
        onChange={e => setMetricValue(e.target.value)}
        placeholder="Metric Value"
      />
      <button onClick={addMetric}>Add Metric</button>
      <ul>
        {metrics.map(metric => (
          <li key={metric._id}>
            {metric.name}: {metric.value} (Timestamp: {new Date(metric.timestamp).toLocaleString()})
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
``

**B. Create the Dockerfile for Frontend** 

``dockerfile
FROM node:14 as build

WORKDIR /app

COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
``

**Step 3: Set Up Prometheus and Grafana**

**A. Create Prometheus Configuration Create a prometheus directory and a prometheus.yml file inside it:**

``bash
mkdir ../prometheus
cd ../prometheus
``

**Create the prometheus.yml configuration file:**

``yaml
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'nodejs-backend'
    static_configs:
      - targets: ['backend:5000']
``

**Step 4: Set Up Docker Compose**

**A. Create the docker-compose.yml file**

``yaml
version: '3.8'
services:
  backend:
    build:
      context: ./backend
    ports:
      - "5000:5000"
    env_file:
      - ./backend/.env
    depends_on:
      - mongo

  frontend:
    build:
      context: ./frontend
    ports:
      - "3000:80"

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    depends_on:
      - prometheus

volumes:
  mongo-data:
``

**Step 5: Running the Application Navigate to the root of your project directory (where docker-compose.yml is located):**

``bash
cd cloud-native-monitoring
Run Docker Compose:
``

``bash
docker-compose up --build
Access the applications:

Frontend: http://localhost:3000
Backend: http://localhost:5000 (API)
Prometheus: http://localhost:9090
Grafana: http://localhost:3001
``

**Step 6: Setting Up Grafana Log in to Grafana:**

The default username and password are both admin.
Change the password when prompted.
Add Prometheus as a data source:

``Go to Configuration (the gear icon) → Data Sources → Add Data Source → Choose Prometheus.
Set the URL to http://prometheus:9090 and click Save & Test.
Create a Dashboard:
Create a new dashboard to visualize the metrics you are collecting.
``

**Step 7: Testing the Application**

``Open your browser and navigate to http://localhost:3000.
You should see the Cloud Native Monitoring Dashboard where you can add and view metrics.
Visit http://localhost:9090 to see Prometheus collecting metrics.
Go to http://localhost:3001 to access Grafana and visualize your metrics.
``

**Step 8: Enhancements**
Once the basic application is running, consider adding:

Alerting: Use Prometheus alerting rules to notify you when certain metrics exceed thresholds.
Detailed dashboards in Grafana to visualize different metrics over time.
Authentication: Secure your backend and Grafana with authentication mechanisms.
Container Metrics: Collect metrics from the Docker containers themselves using cAdvisor.

**Conclusion**
You now have a simple Cloud Native Monitoring Application set up using the MERN stack along with Prometheus and Grafana. This project serves as a foundation to expand and implement more advanced features. Let me know if you have any questions or need further assistance!
