# Health-app
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<title>Simple Health Tracker</title>
<style>
  /* Reset and base */
  * {
    box-sizing: border-box;
  }
  body {
    margin: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #85d8ce, #3a94d6);
    color: #222;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding: 10px;
    min-height: 100vh;
  }
  #app {
    background: #fff;
    width: 100%;
    max-width: 350px;
    border-radius: 12px;
    padding: 20px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.15);
    max-height: 600px;
    overflow-y: auto;
  }
  h1 {
    text-align: center;
    color: #07689f;
    margin-bottom: 10px;
    font-weight: 700;
  }
  h2 {
    color: #055a8c;
    margin-bottom: 8px;
  }
  .section {
    margin-bottom: 24px;
  }
  label {
    font-weight: 600;
    display: block;
    margin-bottom: 6px;
  }
  input[type=number] {
    width: 100%;
    padding: 8px 10px;
    font-size: 1rem;
    border-radius: 8px;
    border: 1.5px solid #3a94d6;
    transition: border-color 0.3s;
  }
  input[type=number]:focus {
    border-color: #07689f;
    outline: none;
  }
  button {
    width: 100%;
    background: #3a94d6;
    color: white;
    font-weight: 700;
    font-size: 1.1rem;
    padding: 12px 0;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    box-shadow: 0 4px 12px rgba(58,148,214,0.6);
    transition: background-color 0.3s ease;
  }
  button:hover {
    background: #07689f;
  }
  .progress-bar {
    background: #eee;
    border-radius: 12px;
    overflow: hidden;
    height: 18px;
    margin-top: 6px;
  }
  .progress-bar-fill {
    height: 100%;
    background: #3a94d6;
    width: 0;
    transition: width 0.5s ease-in-out;
  }
  .progress-text {
    text-align: right;
    font-weight: 600;
    font-size: 0.875rem;
    margin-top: 4px;
    color: #044a75;
  }
  #summary {
    background: #e7f0fa;
    border-radius: 12px;
    padding: 15px;
    box-shadow: inset 0 0 15px rgba(58,148,214,0.3);
  }
  #summary p {
    margin: 6px 0;
    font-weight: 600;
    color: #0b5489;
  }
  @media (max-width: 360px) {
    #app {
      padding: 15px;
    }
    button {
      font-size: 1rem;
      padding: 10px 0;
    }
  }
</style>
</head>
<body>
<div id="app" role="main" aria-label="Health Tracker App">

  <h1>Health Tracker</h1>

  <section class="section" id="water-section">
    <h2>💧 Water Intake (ml)</h2>
    <label for="water-input">Add water (ml):</label>
    <input type="number" id="water-input" min="0" step="50" placeholder="e.g. 250" aria-describedby="water-desc" />
    <button id="water-add-btn">Add Water</button>
    <div class="progress-bar" aria-label="Water intake progress">
      <div class="progress-bar-fill" id="water-progress"></div>
    </div>
    <div class="progress-text" id="water-progress-text">0 / 2000 ml</div>
  </section>

  <section class="section" id="steps-section">
    <h2>🚶 Steps Count</h2>
    <label for="steps-input">Enter steps today:</label>
    <input type="number" id="steps-input" min="0" step="100" placeholder="e.g. 5000" aria-describedby="steps-desc" />
    <button id="steps-set-btn">Set Steps</button>
    <p id="steps-display">Steps today: 0</p>
  </section>

  <section class="section" id="weight-section">
    <h2>⚖️ Weight Tracking (kg)</h2>
    <label for="weight-input">Enter your weight:</label>
    <input type="number" id="weight-input" min="0" step="0.1" placeholder="e.g. 68.5" aria-describedby="weight-desc" />
    <button id="weight-add-btn">Add Weight</button>
    <p id="weight-display">Current weight: N/A</p>
  </section>

  <section class="section" id="summary-section">
    <h2>📊 Summary</h2>
    <div id="summary" aria-live="polite">
      <p>Water Intake: 0 / 2000 ml</p>
      <p>Steps Taken: 0</p>
      <p>Latest Weight: N/A</p>
    </div>
  </section>

</div>

<script>
  (function(){
    const waterGoal = 2000; // ml daily goal

    // Gather elements
    const waterInput = document.getElementById('water-input');
    const waterAddBtn = document.getElementById('water-add-btn');
    const waterProgress = document.getElementById('water-progress');
    const waterProgressText = document.getElementById('water-progress-text');

    const stepsInput = document.getElementById('steps-input');
    const stepsSetBtn = document.getElementById('steps-set-btn');
    const stepsDisplay = document.getElementById('steps-display');

    const weightInput = document.getElementById('weight-input');
    const weightAddBtn = document.getElementById('weight-add-btn');
    const weightDisplay = document.getElementById('weight-display');

    const summary = document.getElementById('summary');

    // Initialize data from localStorage or defaults
    let waterIntake = Number(localStorage.getItem('waterIntake')) || 0;
    let stepsToday = Number(localStorage.getItem('stepsToday')) || 0;
    let weightHistory = JSON.parse(localStorage.getItem('weightHistory')) || [];

    // Update UI functions
    function updateWaterUI(){
      const progressPercent = Math.min((waterIntake / waterGoal) * 100, 100);
      waterProgress.style.width = progressPercent + '%';
      waterProgressText.textContent = `${waterIntake} / ${waterGoal} ml`;
      summary.children[0].textContent = `Water Intake: ${waterIntake} / ${waterGoal} ml`;
    }

    function updateStepsUI(){
      stepsDisplay.textContent = `Steps today: ${stepsToday}`;
      summary.children[1].textContent = `Steps Taken: ${stepsToday}`;
    }

    function updateWeightUI(){
      if(weightHistory.length > 0){
        const latestWeight = weightHistory[weightHistory.length - 1];
        weightDisplay.textContent = `Current weight: ${latestWeight} kg`;
        summary.children[2].textContent = `Latest Weight: ${latestWeight} kg`;
      } else {
        weightDisplay.textContent = 'Current weight: N/A';
        summary.children[2].textContent = 'Latest Weight: N/A';
      }
    }

    // Event listeners
    waterAddBtn.addEventListener('click', () => {
      const inputVal = Number(waterInput.value);
      if(!isNaN(inputVal) && inputVal > 0){
        waterIntake += inputVal;
        localStorage.setItem('waterIntake', waterIntake);
        updateWaterUI();
        waterInput.value = '';
      } else {
        alert('Please enter a valid positive number for water intake.');
      }
    });

    stepsSetBtn.addEventListener('click', () => {
      const inputVal = Number(stepsInput.value);
      if(!isNaN(inputVal) && inputVal >= 0){
        stepsToday = inputVal;
        localStorage.setItem('stepsToday', stepsToday);
        updateStepsUI();
        stepsInput.value = '';
      } else {
        alert('Please enter a valid step count (0 or higher).');
      }
    });

    weightAddBtn.addEventListener('click', () => {
      const inputVal = Number(weightInput.value);
      if(!isNaN(inputVal) && inputVal > 0){
        weightHistory.push(inputVal.toFixed(1));
        localStorage.setItem('weightHistory', JSON.stringify(weightHistory));
        updateWeightUI();
        weightInput.value = '';
      } else {
        alert('Please enter a valid weight greater than 0.');
      }
    });

    // On load UI update
    updateWaterUI();
    updateStepsUI();
    updateWeightUI();

    // Optional: Reset water daily at midnight (simple approach)
    function resetWaterDaily() {
      const lastReset = localStorage.getItem('waterResetDate');
      const today = new Date().toDateString();
      if(lastReset !== today){
        waterIntake = 0;
        localStorage.setItem('waterIntake', waterIntake);
        localStorage.setItem('waterResetDate', today);
        updateWaterUI();
      }
    }
    resetWaterDaily();

  })();
</script>
</body>
</html>
