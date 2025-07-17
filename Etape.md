```bash
docker run --name postgres-logs \
  -e POSTGRES_DB=logs_db \
  -e POSTGRES_USER=logs_user \
  -e POSTGRES_PASSWORD=logs_password \
  -p 5432:5432 \
  -d postgres:15
```

📦 **DB Name**: `logs_db`  
👤 **User**: `logs_user`  
🔑 **Password**: `logs_password`

---

✅ **3. Créer un environnement Python pour le backend**  
💡 Ça t’isole les dépendances.

```bash
cd backend
python3 -m venv venv       # Crée un venv nommé "venv"
source venv/bin/activate   # Active le venv
pip install -r requirements.txt
```

---

✅ **4. Exporter les variables d’environnement**

```bash
export DB_HOST=localhost
export DB_NAME=logs_db
export DB_USER=logs_user
export DB_PASSWORD=logs_password
```

💡 **Sur Windows avec PowerShell** :

```powershell
$env:DB_HOST="localhost"
$env:DB_NAME="logs_db"
$env:DB_USER="logs_user"
$env:DB_PASSWORD="logs_password"
```

---

✅ **5. Lancer le backend**

```bash
python app.py
```

---

✅ **6. Lancer le frontend**  
Ouvre un nouveau terminal :

```bash
cd frontend
python3 -m http.server 8000
```

Accède ensuite à [http://localhost:8000](http://localhost:8000) dans ton navigateur.

---

✅ **7. Vérifier la base de données (optionnel)**  
Tu peux te connecter pour vérifier la table :

```bash
docker exec -it postgres-logs psql -U logs_user -d logs_db
```

Puis créer la table :

```sql
CREATE TABLE logs (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    level VARCHAR(10) NOT NULL,
    message TEXT NOT NULL,
    service VARCHAR(100) DEFAULT 'unknown',
    data JSONB DEFAULT '{}'
);
```

### Copier frontend dans /var/www/html
- Par défaut (dans le fichier default justement), le serveur nginx pointe sur le dossier/var/www/html
- Donc j'y ai copié les fichiers du frontend 