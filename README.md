# app.py 
from flask import Flask, render_template_string, request, redirect, url_for
import sqlite3
import os

app = Flask(__name__)

# Conexión y creación automática de base de datos SQLite
def get_db():
    db_path = os.path.join(os.path.dirname(__file__), 'escuela_cloud.db')
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    return conn

with get_db() as conn:
    conn.execute('''CREATE TABLE IF NOT EXISTS alumnos 
                    (id INTEGER PRIMARY KEY AUTOINCREMENT, 
                     nombre TEXT NOT NULL, 
                     p1 REAL, p2 REAL, p3 REAL)''')
    conn.commit()

# Código HTML integrado directamente en una variable de texto en Python
HTML_UNICO = """
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Boleta CECyTEC 302 - 11NF26</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@1/css/pico.min.css">
</head>
<body class="container" style="padding-top: 20px;">
    <h2>Control de Calificaciones - CECyTEC 302</h2>
    <p><small>Docente: Ing. Humberto G. Aguilera Esparza | Script: 11NF26.py</small></p>
    <hr>
    
    <section>
        <form action="/guardar" method="POST">
            <input type="text" name="nombre" required placeholder="Nombre Completo del Alumno">
            <div class="grid">
                <input type="number" step="0.1" min="0" max="10" name="p1" required placeholder="Parcial 1">
                <input type="number" step="0.1" min="0" max="10" name="p2" required placeholder="Parcial 2">
                <input type="number" step="0.1" min="0" max="10" name="p3" required placeholder="Parcial 3">
            </div>
            <button type="submit" style="background-color: #006341; border: none;">Guardar Alumno</button>
        </form>
    </section>

    <section style="margin-top: 30px;">
        <h3>Boleta Digital Registrada</h3>
        <table>
            <thead>
                <tr>
                    <th>Alumno</th>
                    <th>P1</th>
                    <th>P2</th>
                    <th>P3</th>
                    <th>Promedio</th>
                </tr>
            </thead>
            <tbody>
                {% for a in alumnos %}
                <tr>
                    <td><strong>{{ a.nombre }}</strong></td>
                    <td>{{ a.p1 }}</td>
                    <td>{{ a.p2 }}</td>
                    <td>{{ a.p3 }}</td>
                    <td><strong>{{ "%.1f"|format(a.promedio) }}</strong></td>
                </tr>
                {% else %}
                <tr>
                    <td colspan="5" style="text-align: center;">No hay alumnos registrados todavía.</td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </section>
</body>
</html>
"""

@app.route('/')
def inicio():
    try:
        conn = get_db()
        alumnos = conn.execute('SELECT *, (p1+p2+p3)/3.0 AS promedio FROM alumnos').fetchall()
        conn.close()
    except Exception:
        alumnos = []
    return render_template_string(HTML_UNICO, alumnos=alumnos)

@app.route('/guardar', methods=['POST'])
def guardar():
    try:
        nombre = request.form.get('nombre', 'Sin Nombre').strip()
        p1 = float(request.form.get('p1', 0))
        p2 = float(request.form.get('p2', 0))
        p3 = float(request.form.get('p3', 0))
        
        conn = get_db()
        conn.execute('INSERT INTO alumnos (nombre, p1, p2, p3) VALUES (?, ?, ?, ?)',
                     (nombre, p1, p2, p3))
        conn.commit()
        conn.close()
    except Exception:
        pass
        
    return redirect(url_for('inicio'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
