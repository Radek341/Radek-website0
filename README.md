# Radek-website0
<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Interaktivní Neal.fun styl</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@400;700&display=swap');

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: 'Rubik', sans-serif;
      background: linear-gradient(135deg, #7b2ff7, #f107a3);
      color: white;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2rem;
      text-align: center;
    }

    header h1 {
      font-size: 4rem;
      font-weight: 700;
      margin-bottom: 0.2rem;
      text-shadow: 0 0 5px rgba(0,0,0,0.3);
    }

    header p {
      font-size: 1.3rem;
      opacity: 0.8;
      margin-bottom: 3rem;
    }

    .content {
      background: rgba(255,255,255,0.1);
      border-radius: 1rem;
      padding: 2rem 3rem;
      max-width: 600px;
      box-shadow: 0 0 20px rgba(255,255,255,0.2);
    }

    .content h2 {
      font-size: 2.5rem;
      margin-bottom: 1rem;
    }

    .content p {
      font-size: 1.2rem;
      margin-bottom: 2rem;
      min-height: 3em; /* aby místo pro text bylo vždy */
    }

    button {
      background: white;
      color: #7b2ff7;
      border: none;
      border-radius: 9999px;
      padding: 0.8rem 2rem;
      font-size: 1.2rem;
      cursor: pointer;
      font-weight: 700;
      transition: background-color 0.3s ease, color 0.3s ease;
    }

    button:hover {
      background: #f107a3;
      color: white;
    }

    footer {
      margin-top: auto;
      font-size: 0.9rem;
      opacity: 0.6;
    }
  </style>
</head>
<body>
  <header>
    <h1>neal.fun style</h1>
    <p>Jednoduchý a interaktivní design inspirovaný neal.fun</p>
  </header>

  <main>
    <section class="content">
      <h2>Zkus náhodný vtip!</h2>
      <p id="joke">Klikni na tlačítko níže pro zobrazení vtipu.</p>
      <button id="jokeBtn">Zobraz vtip</button>
    </section>
  </main>

  <footer>
    <p>© 2025 Tvoje jméno</p>
  </footer>

  <script>
    const jokes = [
      "Proč programátoři nemají rádi přírodu? Má moc bugů.",
      "Jak se jmenuje pes programátora? Byte.",
      "Proč je JavaScript smutný? Protože má spoustu callbacků.",
      "Co řekl HTML tag CSS? 'Mám styl!'",
      "Jak programátor oslavuje Vánoce? Deklaruje si dárky jako konstanty."
    ];

    const jokeBtn = document.getElementById('jokeBtn');
    const jokeText = document.getElementById('joke');

    jokeBtn.addEventListener('click', () => {
      const randomIndex = Math.floor(Math.random() * jokes.length);
      jokeText.textContent = jokes[randomIndex];
    });
  </script>
</body>
</html>
