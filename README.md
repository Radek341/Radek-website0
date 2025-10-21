# Radek-website0
<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Snowboarding - Sport a zábava</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Rubik:wght@400;700&display=swap');

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: 'Rubik', sans-serif;
      background: linear-gradient(135deg, #1e3c72, #2a5298);
      color: #fff;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2rem;
      text-align: center;
    }

    header {
      margin-bottom: 2rem;
    }

    header h1 {
      font-size: 3.5rem;
      font-weight: 700;
      text-shadow: 2px 2px 5px rgba(0,0,0,0.7);
    }

    header p {
      font-size: 1.3rem;
      opacity: 0.9;
      margin-top: 0.5rem;
    }

    main {
      max-width: 700px;
      background: rgba(255,255,255,0.1);
      border-radius: 1rem;
      padding: 2rem 3rem;
      box-shadow: 0 0 15px rgba(255,255,255,0.15);
      margin-bottom: 3rem;
    }

    main h2 {
      font-size: 2.5rem;
      margin-bottom: 1rem;
    }

    main p {
      font-size: 1.2rem;
      line-height: 1.6;
      margin-bottom: 1.5rem;
    }

    main img {
      max-width: 100%;
      border-radius: 1rem;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
      margin-bottom: 1.5rem;
    }

    section.coin-flip {
      background: rgba(255,255,255,0.15);
      padding: 2rem 3rem;
      border-radius: 1rem;
      max-width: 400px;
      box-shadow: 0 0 20px rgba(255,255,255,0.25);
    }

    section.coin-flip h2 {
      font-size: 2rem;
      margin-bottom: 1.5rem;
    }

    #result {
      font-size: 2rem;
      margin: 1.5rem 0;
      font-weight: 700;
      color: #f7d154;
      text-shadow: 1px 1px 3px rgba(0,0,0,0.7);
    }

    button {
      background: #f7d154;
      color: #2a5298;
      border: none;
      border-radius: 9999px;
      padding: 0.8rem 2rem;
      font-size: 1.2rem;
      cursor: pointer;
      font-weight: 700;
      transition: background-color 0.3s ease, color 0.3s ease;
      box-shadow: 0 0 10px rgba(247, 209, 84, 0.7);
    }

    button:hover {
      background: #d4b91e;
      color: white;
      box-shadow: 0 0 15px rgba(212, 185, 30, 0.9);
    }

    footer {
      margin-top: auto;
      font-size: 0.9rem;
      opacity: 0.7;
    }
  </style>
</head>
<body>

  <header>
    <h1>Snowboarding</h1>
    <p>Sport, adrenalin a zábava na sněhu</p>
  </header>

  <main>
    <h2>Co je snowboarding?</h2>
    <p>Snowboarding je zimní sport, při kterém jezdec sjíždí sněhový svah na speciální prkně. Spojuje v sobě rychlost, akrobacii a kontakt s přírodou.</p>
    <img src="https://images.unsplash.com/photo-1517836357463-d25dfeac3438?auto=format&fit=crop&w=700&q=80" alt="Snowboarder v akci" />
    
    <h2>Proč snowboarding milujeme?</h2>
    <p>Snowboarding je skvělý způsob, jak trávit čas venku, zlepšovat svou fyzickou kondici a zároveň si užívat svobodu a adrenalin. Navíc je to komunitní sport, kde potkáš spoustu podobně nadšených lidí.</p>
  </main>

  <section class="coin-flip">
    <h2>Hod mincí - vyber si stranu!</h2>
    <button id="flipBtn">Hoď mincí</button>
    <div id="result">Výsledek čeká...</div>
  </section>

  <footer>
    <p>© 2025 Tvoje jméno</p>
  </footer>

  <script>
    const flipBtn = document.getElementById('flipBtn');
    const resultDiv = document.getElementById('result');

    flipBtn.addEventListener('click', () => {
      const sides = ['Orel', 'Panna'];
      const randomSide = sides[Math.floor(Math.random() * sides.length)];
      resultDiv.textContent = `Padlo: ${randomSide}! 🎉`;
    });
  </script>

</body>
</html>
