# Data Mahasiswa Amikom Yogyakarta

Aplikasi web sederhana berbasis Flask untuk mengelola data mahasiswa di Amikom Yogyakarta. Aplikasi ini memungkinkan pengguna untuk melakukan operasi CRUD (Create, Read, Update, Delete) pada data mahasiswa.

## Fitur

- Menampilkan daftar mahasiswa dengan detail (NIM, Nama, Asal).
- Menambah data mahasiswa baru.
- Mengedit data mahasiswa yang sudah ada.
- Menghapus data mahasiswa.

## Prasyarat

Pastikan perangkat Anda sudah terinstal perangkat lunak berikut:

- Python 3.14
- pip (pengelola paket Python)
- XAMPP
- MySQL Server
- Flask dan pustaka terkait

## Instalasi

1. Clone repositori ini:

   ```bash
   git clone https://github.com/your-username/your-repository-name.git
   cd your-repository-name
   ```

2. Buat virtual environment (opsional tapi disarankan):

   ```bash
   python -m venv venv
   source venv/bin/activate # Di Windows, gunakan `venv\Scripts\activate`
   ```

3. Instal paket Python yang dibutuhkan:

   ```bash
   pip install flask mysql-connector-python
   ```

4. Atur database MySQL:

   - Masuk ke MySQL dan buat database:
     ```sql
     CREATE DATABASE db_kuliah;
     ```
   - Gunakan skema berikut untuk tabel `db_kuliah`:
     ```sql
     CREATE TABLE db_kuliah (
         nim VARCHAR(20) PRIMARY KEY,
         nama VARCHAR(100),
         asal VARCHAR(100)
     );
     ```

5. Perbarui file `app.py` dengan kredensial MySQL Anda jika diperlukan:

   ```python
   db = mysql.connector.connect(
       host="localhost",
       user="your-username",
       passwd="your-password",
       database="db_kuliah"
   )
   ```

## Menjalankan Aplikasi

1. Jalankan aplikasi Flask:

   ```bash
   python app.py
   ```

2. Buka browser Anda dan akses:

   ```
   http://127.0.0.1:5000/
   ```

## Menjalankan dengan XAMPP

1. **Instalasi XAMPP**
   Pastikan Anda telah mengunduh dan menginstal XAMPP di perangkat Anda. Pastikan modul **Apache** dan **MySQL** aktif melalui XAMPP Control Panel.

2. **Konfigurasi Database**
   - Buka phpMyAdmin di browser Anda melalui `http://localhost/phpmyadmin`.
   - Buat database dengan nama `db_kuliah`.
   - Jalankan skrip berikut di tab SQL:
     ```sql
     CREATE TABLE db_kuliah (
         nim VARCHAR(20) PRIMARY KEY,
         nama VARCHAR(100),
         asal VARCHAR(100)
     );
     ```

3. **Konfigurasi Flask untuk XAMPP**
   - Pastikan kredensial di file `app.py` sesuai:
     ```python
     db = mysql.connector.connect(
         host="127.0.0.1",
         user="root",
         passwd="",
         database="db_kuliah"
     )
     ```

4. **Menjalankan Aplikasi**
   - Jalankan aplikasi menggunakan perintah berikut di terminal:
     ```bash
     python app.py
     ```
   - Akses aplikasi di browser Anda melalui `http://127.0.0.1:5000/`.

## Catatan Penting

- Pastikan semua service (Apache dan MySQL) sudah berjalan sebelum menjalankan aplikasi.
- Periksa koneksi database sebelum menjalankan operasi CRUD.
- Semua file HTML harus berada dalam folder `templates`.
- Semua asset (CSS, JS, images, fonts) harus berada dalam folder `static`.
- Gunakan virtual environment untuk menghindari konflik dependensi.

## Troubleshooting

1. **Jika terjadi error koneksi database:**
   - Periksa service MySQL sudah berjalan.
   - Periksa kredensial database (username dan password).
   - Pastikan database `db_kuliah` sudah dibuat.

2. **Jika template tidak ditemukan:**
   - Pastikan struktur folder sudah benar.
   - Periksa nama file template sesuai dengan yang dipanggil di `render_template()`.

3. **Jika static files tidak termuat:**
   - Periksa struktur folder `static`.
   - Pastikan path di `url_for()` sudah benar.

## Struktur Proyek

```
.
├── app.py                 # Aplikasi Flask utama
├── templates/             # Template HTML
│   ├── index.html         # Template halaman utama
│   ├── add.html           # Template halaman tambah data mahasiswa
│   └── edit.html          # Template halaman edit data mahasiswa
├── static/                # File statis
│   ├── style.css          # File gaya
│   └── amikom.jpeg        # Gambar logo
└── README.md              # Dokumentasi proyek
```

## Cuplikan Kode

### app.py
```python
from flask import Flask, render_template, request, redirect, url_for
import mysql.connector

app = Flask(__name__)

# Koneksi ke database
db = mysql.connector.connect(
    host="localhost",
    user="root",
    passwd="",
    database="db_kuliah"
)

# Home page
@app.route('/')
def index():
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM db_kuliah")
    mahasiswa = cursor.fetchall()
    return render_template('index.html', mahasiswa=mahasiswa)

# Tambah data
@app.route('/add', methods=['GET', 'POST'])
def add():
    if request.method == 'POST':
        nim = request.form['nim']
        nama = request.form['nama']
        asal = request.form['asal']
        cursor = db.cursor()
        cursor.execute("INSERT INTO db_kuliah (nim, nama, asal) VALUES (%s, %s, %s)", (nim, nama, asal))
        db.commit()
        return redirect(url_for('index'))
    return render_template('add.html')

# Edit data
@app.route('/edit/<nim>', methods=['GET', 'POST'])
def edit(nim):
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM db_kuliah WHERE nim = %s", (nim,))
    mahasiswa = cursor.fetchone()
    if request.method == 'POST':
        nama = request.form['nama']
        asal = request.form['asal']
        cursor = db.cursor()
        cursor.execute("UPDATE db_kuliah SET nama = %s, asal = %s WHERE nim = %s", (nama, asal, nim))
        db.commit()
        return redirect(url_for('index'))
    return render_template('edit.html', mahasiswa=mahasiswa)

# Hapus data
@app.route('/delete/<nim>')
def delete(nim):
    cursor = db.cursor()
    cursor.execute("DELETE FROM db_kuliah WHERE nim = %s", (nim,))
    db.commit()
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```

### index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Amikom Yogyakarta</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <header>
        <img src="{{ url_for('static', filename='amikom.jpeg') }}" alt="Logo" class="logo">
        <h1>Data Mahasiswa Amikom Yogyakarta</h1>
    </header>
    <a href="{{ url_for('add') }}" class="button">Tambah Data</a>
    <table>
        <thead>
            <tr>
                <th>NIM</th>
                <th>Nama</th>
                <th>Asal</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            {% for mhs in mahasiswa %}
            <tr>
                <td>{{ mhs.nim }}</td>
                <td>{{ mhs.nama }}</td>
                <td>{{ mhs.asal }}</td>
                <td>
                    <a href="{{ url_for('edit', old_nim=mhs.nim) }}" class="button-edit">Ubah</a>
                    <a href="{{ url_for('delete', nim=mhs.nim) }}" class="button-delete">Hapus</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>
</html>
```

### add.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Amikom Yogyakarta</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <header>
        <img src="{{ url_for('static', filename='amikom.jpeg') }}" alt="Logo" class="logo">
        <h1>Data Mahasiswa Amikom Yogyakarta</h1>
    </header>
    <form action="{{ url_for('add') }}" method="POST">
        <label for="nim">NIM:</label>
        <input type="text" name="nim" placeholder="NIM" required>
        <label for="nama">Nama:</label>
        <input type="text" name="nama" placeholder="Nama" required>
        <label for="asal">Asal:</label>
        <input type="text" name="asal" placeholder="Asal" required>
        <button type="submit">Submit</button>
    </form>
    <a href="{{ url_for('index') }}" class="button" style="display: inline-block; margin-top: 20px;">Kembali</a>
</body>
</html>
```

### edit.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Amikom Yogyakarta</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <header>
        <img src="{{ url_for('static', filename='amikom.jpeg') }}" alt="Logo" class="logo">
        <h1>Data Mahasiswa Amikom Yogyakarta</h1>
    </header>
    <form action="{{ url_for('edit', old_nim=mahasiswa.nim) }}" method="POST">
        <label for="nim">NIM:</label>
        <input type="text" name="nim" value="{{ mahasiswa.nim }}" required>
        <label for="nama">Nama Lengkap:</label>
        <input type="text" name="nama" value="{{ mahasiswa.nama }}" required>
        <label for="asal">Asal:</label>
        <input type="text" name="asal" value="{{ mahasiswa.asal }}" required>
        <button type="submit">Update</button>
    </form>
    <a href="{{ url_for('index') }}" class="button" style="display: inline-block; margin-top: 20px;">Kembali</a>
</body>
</html>
```

### style.css
```css
:root {
    --primary-color: #4CAF50;
    --secondary-color: #2196F3;
    --accent-color: #FF4081;
    --background-color: #f0f2f5;
    --header-bg: #6200EA;
}

body {
    font-family: 'Segoe UI', Arial, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: var(--background-color);
    color: #333;
    min-height: 100vh;
}

header {
    background-color: var(--header-bg);
    color: white;
    padding: 20px;
    border-radius: 10px;
    margin-bottom: 30px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    display: flex;
    align-items: center;
}

header h1 {
    margin: 0;
    font-size: 28px;
}

table {
    width: 100%;
    background-color: white;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    margin-top: 20px;
}

table th {
    background-color: var(--header-bg);
    color: white;
    padding: 15px;
    font-weight: 500;
}

table td {
    padding: 12px 15px;
    border-bottom: 1px solid #eee;
}

table tr:hover {
    background-color: #f5f5f5;
}

.button {
    background-color: var(--secondary-color);
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    text-decoration: none;
    transition: all 0.3s ease;
    font-weight: 500;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    display: inline-block;
}

.button:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    opacity: 0.9;
}

.button-edit {
    background-color: var(--primary-color);
    color: white;
    padding: 8px 16px;
    margin-right: 5px;
    border-radius: 5px;
    text-decoration: none;
    transition: all 0.3s ease;
    display: inline-block;
}

.button-delete {
    background-color: var(--accent-color);
    color: white;
    padding: 8px 16px;
    border-radius: 5px;
    text-decoration: none;
    transition: all 0.3s ease;
    display: inline-block;
}

.button-edit:hover, .button-delete:hover {
    opacity: 0.9;
    transform: translateY(-2px);
}

form {
    background-color: white;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    max-width: 500px;
    margin: 0 auto;
}

input[type="text"] {
    width: 100%;
    padding: 12px;
    margin: 8px 0;
    border: 2px solid #ddd;
    border-radius: 5px;
    box-sizing: border-box;
    transition: border-color 0.3s ease;
}

input[type="text"]:focus {
    border-color: var(--secondary-color);
    outline: none;
}

button[type="submit"] {
    background-color: var(--primary-color);
    color: white;
    padding: 12px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    width: 100%;
    font-size: 16px;
    margin-top: 10px;
    transition: all 0.3s ease;
}

button[type="submit"]:hover {
    background-color: #43A047;
    transform: translateY(-2px);
}

label {
    font-weight: 500;
    color: #666;
    display: block;
    margin-top: 10px;
}

.logo {
    width: auto; 
    height: 60px; 
    margin-right: 20px;
    border-radius: 10px; 
    border: 2px solid white;
    object-fit: contain; 
    background-color:#eee;  
    padding: 5px; 
}
```
