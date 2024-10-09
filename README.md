# suivis-outillages

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Tool(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    description = db.Column(db.String(200), nullable=False)
    is_borrowed = db.Column(db.Boolean, default=False)

class Borrowing(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    tool_id = db.Column(db.Integer, db.ForeignKey('tool.id'), nullable=False)
    borrower = db.Column(db.String(100), nullable=False)
    borrow_date = db.Column(db.DateTime, nullable=False)
    return_date = db.Column(db.DateTime)



import sys
import os
from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

# Ajouter le répertoire courant au PYTHONPATH
current_dir = os.path.dirname(os.path.abspath(__file__))
sys.path.append(current_dir)
print(f"Répertoire courant ajouté au PYTHONPATH : {current_dir}")

app = Flask(__name__)

# Configurer la base de données SQLite
db_path = os.path.join(current_dir, '../tools.db')
app.config['SQLALCHEMY_DATABASE_URI'] = f'sqlite:///{db_path}'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Afficher le chemin absolu de la base de données pour vérification
print(f"Chemin absolu de la base de données : {db_path}")

# Initialiser SQLAlchemy
db = SQLAlchemy()
db.init_app(app)

# Importer les modèles
try:
    from models import Tool, Borrowing
    print("Importation des modèles réussie")
except ModuleNotFoundError as e:
    print(f"Erreur d'importation : {e}")

# Créer les tables
with app.app_context():
    db.create_all()

# Page d'accueil : Liste des outils
@app.route('/')
def index():
    tools = Tool.query.all()  # Récupérer tous les outils
    return render_template('index.html', tools=tools)

# Ajouter un nouvel outil
@app.route('/add', methods=['POST'])
def add_tool():
    name = request.form['name']
    description = request.form['description']
    new_tool = Tool(name=name, description=description)
    db.session.add(new_tool)
    db.session.commit()
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)


<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestion des Outillages</title>
    <link rel="stylesheet" href="/static/style.css">
</head>
<body>
    <h1>Gestion des Outillages</h1>

    <!-- Formulaire pour ajouter un outil -->
    <h2>Ajouter un nouvel outil</h2>
    <form action="/add" method="POST">
        <input type="text" name="name" placeholder="Nom de l'outil" required>
        <input type="text" name="description" placeholder="Description de l'outil" required>
        <button type="submit">Ajouter l'outil</button>
    </form>

    <!-- Liste des outils -->
    <h2>Liste des outils</h2>
    <table>
        <thead>
            <tr>
                <th>Nom</th>
                <th>Description</th>
                <th>Status</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for tool in tools %}
            <tr>
                <td>{{ tool.name }}</td>
                <td>{{ tool.description }}</td>
                <td>{{ 'Emprunté' if tool.is_borrowed else 'Disponible' }}</td>
                <td>
                    {% if not tool.is_borrowed %}
                    <form action="/borrow/{{ tool.id }}" method="POST" style="display:inline;">
                        <input type="text" name="borrower" placeholder="Nom de l'emprunteur" required>
                        <button type="submit">Emprunter</button>
                    </form>
                    {% else %}
                    <form action="/return/{{ tool.id }}" method="POST" style="display:inline;">
                        <button type="submit">Retourner</button>
                    </form>
                    {% endif %}
                    <form action="/delete/{{ tool.id }}" method="POST" style="display:inline;">
                        <button type="submit" onclick="return confirm('Supprimer cet outil ?');">Supprimer</button>
                    </form>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>
</html>

body {
    font-family: Arial, sans-serif;
    margin: 20px;
}

h1, h2 {
    color: #333;
}

form {
    margin-bottom: 20px;
}

input, button {
    padding: 10px;
    margin: 5px;
}

table {
    width: 100%;
    border-collapse: collapse;
}

table, th, td {
    border: 1px solid #ccc;
}

th, td {
    padding: 10px;
    text-align: left;
}

button {
    cursor: pointer;
    background-color: #5cb85c;
    color: white;
    border: none;
    padding: 10px 15px;
}

button:hover {
    background-color: #4cae4c;
}
