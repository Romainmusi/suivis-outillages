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
