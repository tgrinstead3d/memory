<script lang="ts">
  import { onMount } from 'svelte';

  // Card symbols for our 5x4 grid (we need 20 cards = 10 pairs)
  const symbols = ['🍎', '🍌', '🍇', '🍓', '🍍', '🍉', '🥭', '🍋', '🍒', '🥝'];
  
  // Game state
  let cards: { id: number; symbol: string; flipped: boolean; matched: boolean }[] = [];
  let flippedCards: number[] = [];
  let matchedPairs = 0;
  let moves = 0;
  let gameStarted = false;
  let gameCompleted = false;
  let timer = 0;
  let timerInterval: ReturnType<typeof setInterval> | null = null;

  // Initialize the game
  function initGame() {
    // Reset game state
    matchedPairs = 0;
    moves = 0;
    flippedCards = [];
    gameCompleted = false;
    timer = 0;
    
    if (timerInterval) {
      clearInterval(timerInterval);
      timerInterval = null;
    }

    // Create pairs of cards
    const cardSymbols = [...symbols, ...symbols];
    
    // Shuffle the cards
    const shuffledSymbols = cardSymbols.sort(() => Math.random() - 0.5);
    
    // Create card objects
    cards = shuffledSymbols.map((symbol, index) => ({
      id: index,
      symbol,
      flipped: false,
      matched: false
    }));
    
    gameStarted = true;
    
    // Start the timer
    timerInterval = setInterval(() => {
      timer++;
    }, 1000);
  }

  // Handle card click
  function flipCard(id: number) {
    // Ignore clicks if two cards are already flipped or the clicked card is already flipped/matched
    if (flippedCards.length >= 2 || cards[id].flipped || cards[id].matched || gameCompleted) {
      return;
    }

    // Flip the card
    cards[id].flipped = true;
    flippedCards = [...flippedCards, id];

    // Check for a match if two cards are flipped
    if (flippedCards.length === 2) {
      moves++;
      
      const [firstCardId, secondCardId] = flippedCards;
      
      if (cards[firstCardId].symbol === cards[secondCardId].symbol) {
        // Match found
        setTimeout(() => {
          cards[firstCardId].matched = true;
          cards[secondCardId].matched = true;
          matchedPairs++;
          flippedCards = [];
          cards = [...cards];
          
          // Check if game is completed
          if (matchedPairs === 10) {
            gameCompleted = true;
            if (timerInterval) {
              clearInterval(timerInterval);
              timerInterval = null;
            }
          }
        }, 500);
      } else {
        // No match, flip cards back
        setTimeout(() => {
          cards[firstCardId].flipped = false;
          cards[secondCardId].flipped = false;
          flippedCards = [];
          cards = [...cards];
        }, 1000);
      }
    }
  }

  // Format time display
  function formatTime(seconds: number): string {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  }

  // Initialize game on mount
  onMount(() => {
    initGame();
    return () => {
      if (timerInterval) clearInterval(timerInterval);
    };
  });
</script>

<main>
  <div class="game-container">
    <h1>Memory Game</h1>
    
    <div class="memory-grid">
      {#each cards as card (card.id)}
        <div 
          class="card {card.flipped ? 'flipped' : ''} {card.matched ? 'matched' : ''}"
          on:click={() => flipCard(card.id)}
          on:keydown={(e) => e.key === 'Enter' && flipCard(card.id)}
          role="button"
          tabindex="0"
          aria-label="Memory card {card.flipped ? 'showing ' + card.symbol : 'face down'}"
          aria-pressed={card.flipped}
        >
          <div class="card-inner">
            <div class="card-front"></div>
            <div class="card-back">{card.symbol}</div>
          </div>
        </div>
      {/each}
    </div>
    
    <div class="stats">
      <p>Time: {formatTime(timer)} | Moves: {moves}</p>
      {#if gameCompleted}
        <p>🎉 Congratulations! You've completed the game! 🎉</p>
      {/if}
    </div>
    
    <div class="controls">
      <button on:click={initGame}>New Game</button>
    </div>
  </div>
</main>
