<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';

// Initialize variables and starting state
const suits = ["H", "D", "C", "S"];
const numbers = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"];
const jackb = ['A', 'J', 'Q', 'K'];
const flush_list = ["S,S,S,S,S", "C,C,C,C,C", "D,D,D,D,D", "H,H,H,H,H"];
const straight_list = [
  ["A", "2", "3", "4", "5"],
  ["2", "3", "4", "5", "6"], 
  ["3", "4", "5", "6", "7"], 
  ["4", "5", "6", "7", "8"], 
  ["5", "6", "7", "8", "9"], 
  ["6", "7", "8", "9", "T"],
  ["7", "8", "9", "T", "J"],
  ["8", "9", "T", "J", "Q"],
  ["9", "T", "J", "Q", "K"],
  ["T", "J", "Q", "K", "A"]
];

let deck = ref<string[]>([]);
const hand = ref<string[]>([]);
const dealt_hand = ref<string[]>([]);
const holds = ref<boolean[]>([false, false, false, false, false]);

// Game statistics
const royalFlushes = ref(0);
const straightFlushes = ref(0);
const fourKinds = ref(0);
const fullHouses = ref(0);
const flushes = ref(0);
const straights = ref(0);
const threeKinds = ref(0);
const twoPairs = ref(0);
const pairs = ref(0);
const nothing = ref(0);
const totalWon = ref(0);
const coins = ref(5000);

// UI state
const handHolder = ref("");
const gamePhase = ref("pre-deal"); // pre-deal, choosing, results
const showHandList = ref(false);
const winningsMessage = ref("");

// Helper functions
function makeDeck() {
  deck.value = [];
  numbers.forEach(n => {
    suits.forEach(s => {
      deck.value.push(n + s);
    });
  });
}

// Functions to check hands
function royal_flush(numbers1: string[], suits1: string[]) {
  if (flush_list.includes(suits1.join()) && straight_list[9].sort().join() == numbers1.sort().join()) {
    return true;
  }
  return false;
}

function straight_flush(numbers1: string[], suits1: string[]) {
  if (flush_list.includes(suits1.join())) {
    for (let index = 0; index < 9; index++) {
      if (straight_list[index].sort().join() == numbers1.sort().join()) {
        return true;
      }
    }
  }
  return false;
}

function four_kind(numbers1: string[]) {
  let counter = 0;
  let holder: number[] = [];
  numbers1.forEach(n => {
    counter = 0;
    for (let i = 0; i < 5; i++) {
      if (numbers1[i] == n) {
        counter = counter + 1;
      }
    }
    holder.push(counter);
  });
  if (holder.sort((a, b) => a - b).pop() == 4) {
    return true;
  }
  return false;
}

function full_house(numbers1: string[]) {
  let counter = 0;
  let holder: number[] = [];
  numbers1.forEach(n => {
    counter = 0;
    for (let i = 0; i < 5; i++) {
      if (numbers1[i] == n) {
        counter = counter + 1;
      }
    }
    holder.push(counter);
  });
  holder.sort((a, b) => a - b);
  if (holder[holder.length - 1] == 3 && holder[0] == 2) {
    return true;
  }
  return false;
}

function flush(suits1: string[]) {
  if (flush_list.includes(suits1.join())) {
    return true;
  }
  return false;
}

function straight(numbers1: string[]) {
  for (let i = 0; i < straight_list.length; i++) {
    if (straight_list[i].sort().join() == numbers1.sort().join()) {
      return true;
    }
  }
  return false;
}

function three_kind(numbers1: string[]) {
  let counter = 0;
  let holder: number[] = [];
  numbers1.forEach(n => {
    counter = 0;
    for (let i = 0; i < 5; i++) {
      if (numbers1[i] == n) {
        counter = counter + 1;
      }
    }
    holder.push(counter);
  });
  if (holder.sort((a, b) => a - b).pop() == 3) {
    return true;
  }
  return false;
}

function two_pair(numbers1: string[]) {
  let counter = 0;
  let holder: number[] = [];
  numbers1.forEach(n => {
    counter = 0;
    for (let i = 0; i < 5; i++) {
      if (numbers1[i] == n) {
        counter = counter + 1;
      }
    }
    holder.push(counter);
  });
  if ((holder.filter(m => m == 2)).length == 4) {
    return true;
  }
  return false;
}

function jacks_or_better(numbers1: string[]) {
  let counter = 0;
  let holder: number[] = [];
  jackb.forEach(n => {
    counter = 0;
    for (let i = 0; i < 5; i++) {
      if (numbers1[i] == n) {
        counter = counter + 1;
      }
    }
    holder.push(counter);
  });
  if ((holder.filter(m => m == 2)).length == 1) {
    return true;
  }
  return false;
}

// Game actions
function dealHand() {
  resetCounts();
  gamePhase.value = "choosing";
  makeDeck();
  
  // Deal a hand
  dealt_hand.value = [];
  let a;
  for (let index = 0; index < 5; index++) {
    a = deck.value[(Math.floor(Math.random() * deck.value.length))];
    dealt_hand.value.push(a);
    deck.value = deck.value.filter(card => card !== a);
  }
  
  // Reset holds
  holds.value = [false, false, false, false, false];
  
  // Update coins
  coins.value -= 50;
}

function resetCounts() {
  royalFlushes.value = 0;
  straightFlushes.value = 0;
  fourKinds.value = 0;
  fullHouses.value = 0;
  flushes.value = 0;
  straights.value = 0;
  threeKinds.value = 0;
  twoPairs.value = 0;
  pairs.value = 0;
  nothing.value = 0;
  totalWon.value = 0;
}

function toggleHold(index: number) {
  holds.value[index] = !holds.value[index];
}

// Add this new function to format the hand result for display
function formatHandResult(hand: string[], handType: string, coinsWon: number) {
  let result = {
    cards: hand,
    handType: handType,
    coinsWon: coinsWon
  };
  return JSON.stringify(result);
}

// Modify the checkHandValue function to use the new format
function checkHandValue(hand: string[], numbers1: string[], suits1: string[]) {
  if (royal_flush(numbers1, suits1)) {
    handHolder.value += formatHandResult(hand, "Royal Flush", 250);
    royalFlushes.value++;
    totalWon.value += 250;
    coins.value += 250;
  } else if (straight_flush(numbers1, suits1)) {
    handHolder.value += formatHandResult(hand, "Straight Flush", 40);
    straightFlushes.value++;
    totalWon.value += 40;
    coins.value += 40;
  } else if (four_kind(numbers1)) {
    handHolder.value += formatHandResult(hand, "4 of a kind", 20);
    fourKinds.value++;
    totalWon.value += 20;
    coins.value += 20;
  } else if (full_house(numbers1)) {
    handHolder.value += formatHandResult(hand, "Full House", 9);
    fullHouses.value++;
    totalWon.value += 9;
    coins.value += 9;
  } else if (flush(suits1)) {
    handHolder.value += formatHandResult(hand, "Flush", 6);
    flushes.value++;
    totalWon.value += 6;
    coins.value += 6;
  } else if (straight(numbers1)) {
    handHolder.value += formatHandResult(hand, "Straight", 5);
    straights.value++;
    totalWon.value += 5;
    coins.value += 5;
  } else if (three_kind(numbers1)) {
    handHolder.value += formatHandResult(hand, "3 of a kind", 3);
    threeKinds.value++;
    totalWon.value += 3;
    coins.value += 3;
  } else if (two_pair(numbers1)) {
    handHolder.value += formatHandResult(hand, "Two Pair", 2);
    twoPairs.value++;
    totalWon.value += 2;
    coins.value += 2;
  } else if (jacks_or_better(numbers1)) {
    handHolder.value += formatHandResult(hand, "Jacks or better", 1);
    pairs.value++;
    totalWon.value += 1;
    coins.value += 1;
  } else {
    nothing.value++;
  }
  
  // Add a separator between hand results
  handHolder.value += "|||";
}

// Add computed property to parse the hand results
const parsedHandResults = computed(() => {
  if (!handHolder.value) return [];
  
  // Remove the initial heading
  let results = handHolder.value.replace("<h3>Hits</h3><h3>", "").split("|||");
  
  // Filter out empty strings and parse JSON
  return results
    .filter(result => result.trim().length > 0)
    .map(result => {
      try {
        return JSON.parse(result);
      } catch (e) {
        return null;
      }
    })
    .filter(result => result !== null);
});

// Modify drawCards to use the new format
function drawCards() {
  gamePhase.value = "results";
  handHolder.value = "<h3>Hits</h3><h3>";
  
  // Run 50 simulations
  for (let index = 0; index < 50; index++) {
    makeDeck();
    
    // Remove the dealt hand from deck
    for (let j = 0; j < 5; j++) {
      deck.value = deck.value.filter(d => d !== dealt_hand.value[j]);
    }
    
    // Create new hand with held cards
    hand.value = [];
    for (let i = 0; i < 5; i++) {
      if (holds.value[i]) {
        hand.value.push(dealt_hand.value[i]);
      }
    }
    
    // Fill in the rest with random cards
    let rest = 5 - hand.value.length;
    for (let i = 0; i < rest; i++) {
      let b = deck.value[Math.floor(Math.random() * deck.value.length)];
      hand.value.push(b);
      deck.value = deck.value.filter(e => e !== b);
    }
    
    let numbers1: string[] = [];
    let suits1: string[] = [];
    
    hand.value.forEach(card => {
      numbers1.push(card[0]);
      suits1.push(card[1]);
    });
    
    // Check hand values
    checkHandValue(hand.value, numbers1, suits1);
  }
  
  handHolder.value += "</h3>";
  showHandList.value = true;
  winningsMessage.value = "Total Coins won: " + totalWon.value;
}

function claimWinnings() {
  gamePhase.value = "pre-deal";
  showHandList.value = false;
  winningsMessage.value = "";
}

onMounted(() => {
  makeDeck();
});

// Computed properties for styling
const getHandStatStyle = (count: number) => {
  return count > 0 ? 'color: red;' : 'color: blue;';
};

// Add this function to convert your card notation to the library format
function getCardImageUrl(card: string) {
  if (!card) return '';
  
  const value = card.slice(0, -1); // Get the value part of the card
  const suit = card.slice(-1); // Get the suit part of the card
  
  // Map '10' to '0' for the API
  const valueMap: Record<string, string> = {
    '2': '2',
    '3': '3',
    '4': '4',
    '5': '5',
    '6': '6',
    '7': '7',
    '8': '8',
    '9': '9',
    '10': '0',  // Map '10' to '0'
    'J': 'J',
    'Q': 'Q',
    'K': 'K',
    'A': 'A'
  };
  
  return `https://deckofcardsapi.com/static/img/${valueMap[value]}${suit}.png`;
}
</script>

<template>
  <div class="container">
    <h1 id="header">~~~~~50 Hand Video Poker~~~~~</h1>
    <h3 v-if="gamePhase === 'choosing'" id="choose">Click cards to hold then press DRAW</h3>
    
    <div class="row">
      <div id="count_area" class="col-md-4">
        <h3 style="text-align: center; margin-bottom: 15px; color: #007bff; border-bottom: 1px solid #dee2e6; padding-bottom: 10px;">
          Hand Statistics
        </h3>
        <div 
          v-for="(count, type) in {
            'Royal Flush': royalFlushes,
            'Straight Flush': straightFlushes,
            '4 of a Kind': fourKinds,
            'Full House': fullHouses,
            'Flush': flushes,
            'Straight': straights,
            '3 of a Kind': threeKinds,
            'Two Pair': twoPairs,
            'Jacks or better': pairs
          }" 
          :key="type"
          class="stat-entry"
          :class="{ 'active': count > 0 }"
        >
          <h4 :style="getHandStatStyle(count)">{{ type }}: {{ count }}</h4>
        </div>
        <div v-if="nothing > 0" class="stat-entry">
          <p>Nothing: {{ nothing }}</p>
        </div>
      </div>
      
      <div id="card_area" class="col-md-8">
        <h1 v-if="winningsMessage" id="winnings">{{ winningsMessage }}</h1>
        <p v-if="gamePhase === 'choosing'" id="helper">Click cards to hold then press DRAW</p>
        
        <div class="card-container" v-show="gamePhase !== 'pre-deal'">
          <div 
            v-for="(card, index) in dealt_hand" 
            :key="index" 
            class="card-slot" 
            @click="toggleHold(index)"
            :class="{ 'held': holds[index] }"
          >
            <img :src="getCardImageUrl(card)" :alt="card" class="card-image" />
            <div v-if="holds[index]" class="hold-banner">HOLD</div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="row">
      <div class="col-md-12 text-center">
        <h2 id="coins">Coins: {{ coins }}</h2>
        <button v-if="gamePhase === 'pre-deal'" id="deal" class="btn btn-success" @click="dealHand">DEAL</button>
        <button v-if="gamePhase === 'choosing'" id="draw" class="btn btn-primary" @click="drawCards">DRAW</button>
        <button v-if="gamePhase === 'results'" id="claim" class="btn btn-danger" @click="claimWinnings">CLAIM</button>
      </div>
    </div>
    
    <div v-if="showHandList" id="hand_list">
      <h3 class="hits-header">Winning Hands</h3>
      <div class="hits-container">
        <div v-for="(result, index) in parsedHandResults" :key="index" class="hit-item">
          <div class="hit-cards">
            <div v-for="(card, cardIndex) in result.cards" :key="cardIndex" class="hit-card">
              <img :src="getCardImageUrl(card)" :alt="card" class="hit-card-image" />
            </div>
          </div>
          <div class="hit-details">
            <span class="hit-type">{{ result.handType }}</span>
            <span class="hit-coins">+{{ result.coinsWon }} coins</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
#header {
  text-align: center;
  color: blue;
  margin-bottom: 20px;
  padding: 10px;
  border-bottom: 2px solid #007bff;
}

.row {
  margin-bottom: 30px;
}

#count_area {
  background-color: #f8f9fa;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  min-height: 450px;
}

#card_area {
  margin-top: 30px;
  background-color: #e9ecef;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  min-height: 450px;
}

.card-container {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  gap: 10px;
}

.card-slot {
  position: relative;
  width: 100px;
  height: 150px;
  padding: 5px;
  background-color: transparent;
  border: none;
  transition: transform 0.2s ease;
  cursor: pointer;
}

.card-slot:hover {
  transform: translateY(-5px);
}

.card-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.card-slot.held {
  transform: translateY(-10px);
}

.hold-banner {
  position: absolute;
  bottom: 20px;
  left: 0;
  right: 0;
  background-color: rgba(220, 53, 69, 0.8);
  color: white;
  text-align: center;
  padding: 5px;
  font-weight: bold;
  border-radius: 5px;
  transform: rotate(-5deg);
  width: 80%;
  margin: 0 auto;
}

#coins {
  background-color: #ffc107;
  color: #212529;
  padding: 10px;
  border-radius: 10px;
  display: inline-block;
  margin-right: 20px;
  font-weight: bold;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}

#deal, #draw {
  margin: 20px 10px;
  padding: 10px 30px;
  font-size: 18px;
  font-weight: bold;
}

#claim {
  margin: 10px 0;
  padding: 10px 30px;
  font-size: 18px;
  font-weight: bold;
}

#winnings {
  color: #dc3545;
  background-color: #f8d7da;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 15px;
  font-weight: bold;
}

#helper {
  color: #0c5460;
  background-color: #d1ecf1;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 15px;
}

#hand_list {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 10px;
  margin-top: 20px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  max-height: 600px;
  overflow-y: auto;
}

.hits-header {
  text-align: center;
  color: #007bff;
  margin-bottom: 15px;
  border-bottom: 1px solid #dee2e6;
  padding-bottom: 10px;
}

.hits-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
}

.hit-item {
  background-color: white;
  border-radius: 8px;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  width: 300px;
  margin-bottom: 15px;
}

.hit-cards {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
  gap: 2px;
}

.hit-card {
  width: 50px;
  height: 70px;
}

.hit-card-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 4px;
}

.hit-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px 10px;
  background-color: #e9ecef;
  border-radius: 4px;
}

.hit-type {
  font-weight: bold;
  color: #495057;
}

.hit-coins {
  color: #28a745;
  font-weight: bold;
}

.stat-entry {
  padding: 8px;
  margin: 5px 0;
  border-radius: 5px;
}

.stat-entry.active {
  background-color: #f8d7da;
}

@media (max-width: 768px) {
  .card-slot {
    width: 80px;
    height: 120px;
  }
}

@media (min-width: 992px) {
  .card-slot {
    width: 120px;
    height: 180px;
  }
}
</style>
