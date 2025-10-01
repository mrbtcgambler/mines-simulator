# Provably Fair Mines Simulator

Welcome to the world's first fully functional, provably fair Mines simulator, created for the MrBtcGambler community. This tool is designed to replicate the gameplay and cryptographic logic of the popular casino game "Mines," allowing players to test strategies, understand game mechanics, and learn about the risks of gambling in a safe, offline environment.

This project was developed to promote responsible gaming. Before you risk real money on a new strategy, test it here first!

## Key Features

This simulator is a single, self-contained HTML file packed with powerful features:

* **Provably Fair Core:** Utilizes the exact `HMAC-SHA256` cryptographic algorithm as the real game, ensuring that all results are accurate and verifiable against third-party tools.
* **Manual Play Mode:** Play the game just as you would on the site. Set your bet and mine count, then click tiles one by one and decide when to cash out.
* **Advanced Autobet Mode:** Automate your gameplay to test strategies over thousands of rounds.
    * **Tile Selection:** Pre-select the exact tiles you want the bot to click.
    * **Strategy Configuration:** Set rules for how your bet amount should change after a win or a loss (e.g., reset to base or increase by a percentage).
    * **Turbo Mode:** Run simulations at maximum speed by skipping visual animations, allowing for rapid data collection.
* **Code Mode (Advanced):** Write your own dynamic betting strategies in JavaScript to react to game outcomes in real-time.
* **In-Depth Verification:**
    * Manually set your own `Server Seed`, `Client Seed`, and `Nonce` to replicate specific game outcomes.
    * Use the **Debug Last Game** button to see a detailed, step-by-step breakdown of the cryptographic calculations for the last played game.

## How to Use the Simulator

1.  **Download:** Save the `playableMines.html` file to your computer.
2.  **Open:** Open the file in any modern web browser (like Chrome, Firefox, or Edge). The simulator is fully self-contained and requires no internet connection.

### Manual Play

This mode is for playing the game one round at a time.

1.  Ensure the **"Manual"** toggle is active at the top of the control panel.
2.  Set your desired **Bet Amount** and **Mines** count.
3.  Enter the **Server Seed**, **Client Seed**, and **Nonce** you wish to test, or click **"New Seeds"** for a random set.
4.  Click the green **"Bet"** button. Your balance will be debited.
5.  Click on the tiles on the game grid to reveal them.
6.  If you find gems, the **"Cashout"** button will become active. Click it at any time to end the round and collect your winnings.
7.  If you hit a mine, the game ends, and all mine locations are revealed.

### Autobet Mode

This mode is for testing a specific strategy over many rounds.

1.  Click the **"Auto"** toggle at the top of the control panel.
2.  Set your **Base Bet Amount**, **Mines** count, and the **Number of Bets** to run.
3.  **Select Tiles:** Click on the game grid to choose which tiles the bot should click in every round. Selected tiles will have a yellow border.
4.  **Configure Strategy:** Use the "On Win" and "On Loss" controls to decide if the bet amount should `Reset` to the base or `Increase` by a certain percentage.
5.  **(Optional) Turbo Mode:** For maximum speed, enable the "Turbo Mode" switch.
6.  Click **"Start Autobet"**. The simulation will run automatically. You can click **"Stop Autobet"** at any time.

### Code Mode (Advanced)

This is the most powerful mode, allowing you to create complex, dynamic strategies using JavaScript.

1.  Click the **"Code"** toggle at the top of the control panel.
2.  **Write Your Strategy:** Use the built-in script editor. Your code has access to a simple API to control the betting bot.
3.  Click **"Start Script"**. The simulator will execute your code and run bets based on your logic. A **Script Log** will appear below the game grid to show output from your code.

#### Scripting API

Your script has access to three main components: `config`, `engine`, and `randomFields`.

**1. The `config` Object**

This object holds the parameters for the *next* bet. You define its initial state at the top of your script, and you can change its properties after each bet inside the `engine.onBetPlaced` callback.

* `config.betSize` (number): The amount to wager on the next bet.
* `config.mines` (number): The number of mines for the next bet (1-24).
* `config.fields` (array of numbers): The tile positions the bot should click (e.g., `[0, 8, 16]`).
* `config.numBets` (number): The total number of bets to run.

**2. The `engine` Object**

This object provides the core functionality for your script.

* `engine.log(...args)`: Prints messages, variables, or objects to the **Script Log**. This is your primary tool for debugging.
* `engine.onBetPlaced(async (lastBet, state) => { ... })`: This is the most important function. You pass it a callback function that contains your core strategy logic. This callback is executed *after* each bet is resolved.

**The `onBetPlaced` Callback Parameters:**

* `lastBet` (object): Contains information about the bet that just finished.
    * `lastBet.win` (boolean): `true` if you did not hit a mine, `false` otherwise.
    * `lastBet.betSize` (number): The amount that was wagered.
    * `lastBet.payout` (number): The multiplier for the win (e.g., `1.23`). It will be `0` on a loss.
* `state` (object): Contains the current global state of the simulator.
    * `state.balance` (number): Your current total balance.
    * `state.profit` (number): Your current session profit.

**3. The `randomFields(amount)` Function**

A global helper function that returns an array of a specified `amount` of random, unique tile numbers. This is useful for strategies that involve changing tile selections.

#### Example Script: Martingale Strategy

The editor comes pre-filled with this simple Martingale strategy.

```javascript
// Define initial bet parameters in the config object.
// These can be changed dynamically in the onBetPlaced event.
config.betSize = 0.0001;
config.mines = 5;
config.fields = [0, 1, 2, 3, 4]; // Tiles to click (0-24)
config.numBets = 1000; // Optional: Or run indefinitely

engine.log('Strategy script loaded!');

// This function is called after each bet is resolved.
engine.onBetPlaced(async (lastBet, state) => {
    // state object contains { balance, profit }

    if (lastBet.win) {
        // On win, reset to base bet
        engine.log('Win! Payout:', lastBet.payout.toFixed(2) + 'x', 'Resetting bet.');
        config.betSize = 0.0001; 
    } else {
        // On loss, apply Martingale (2x bet)
        engine.log('Loss! Doubling bet.');
        config.betSize *= 2;
    }

    // You can also change mines or fields for the next bet
    // For example, to pick 5 new random tiles on every bet:
    // config.fields = randomFields(5);
});
