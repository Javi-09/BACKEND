const express = require('express');
const cors = require('cors');
const fetch = require('node-fetch'); // Esto debería funcionar con la versión 2.6.1

const app = express();
const port = 4000;

app.use(cors()); // Permitir CORS

// Ruta para extraer métricas de SonarQube
app.get('/api/measures/component', async (req, res) => {
    const { projectKey, token } = req.query;

    try {
        const response = await fetch(`http://localhost:9000/api/measures/component?component=${projectKey}&metricKeys=ncloc,complexity,cognitive_complexity,classes,functions,duplicated_lines,duplicated_blocks,comment_lines,class_complexity,file_complexity,comment_lines_density,bugs`, {
            headers: {
                Authorization: `Basic ${Buffer.from(`${token}:`).toString('base64')}`,
            },
        });

        if (!response.ok) {
            const errorData = await response.json();
            return res.status(500).json({ error: errorData.errors[0].msg || "Error al obtener métricas" });
        }

        const data = await response.json();
        res.json(data);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

app.listen(port, () => {
    console.log(`Servidor escuchando en http://localhost:${port}`);
});
