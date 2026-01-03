<script setup>
import { ref, reactive } from 'vue';
import { options } from './data/options.js';

const result = reactive({
  weapon: '',
  monster: '',
  equipment: [],
  behavior: [],
  special: []
});

const config = reactive({
  equipmentCount: 1,
  behaviorCount: 1,
  specialCount: 1
});

const getRandomItem = (arr) => {
  return arr[Math.floor(Math.random() * arr.length)];
};

const getRandomItems = (arr, count) => {
  const shuffled = [...arr].sort(() => 0.5 - Math.random());
  return shuffled.slice(0, count);
};

const generateWeapon = () => {
  result.weapon = getRandomItem(options.weapons);
};

const generateMonster = () => {
  result.monster = getRandomItem(options.monsters);
};

const generateEquipment = () => {
  result.equipment = getRandomItems(options.equipmentRestrictions, config.equipmentCount);
};

const generateBehavior = () => {
  result.behavior = getRandomItems(options.behaviorRestrictions, config.behaviorCount);
};

const generateSpecial = () => {
  result.special = getRandomItems(options.specialRestrictions, config.specialCount);
};

const generateChallenge = () => {
  generateWeapon();
  generateMonster();
  generateEquipment();
  generateBehavior();
  generateSpecial();
};
</script>

<template>
  <div class="container">
    <header>
      <h1>MHW 随机挑战生成器</h1>
      <p>May the Sapphire Star light your way!</p>
    </header>

    <button class="generate-btn" @click="generateChallenge">一键生成所有 (Generate All)</button>

    <div class="results">
      <div class="card weapon-card">
        <div class="card-header">
          <h2>武器 (Weapon)</h2>
          <button class="mini-btn" @click="generateWeapon">抽取</button>
        </div>
        <div class="value">{{ result.weapon || '???' }}</div>
      </div>

      <div class="card monster-card">
        <div class="card-header">
          <h2>目标怪物 (Target)</h2>
          <button class="mini-btn" @click="generateMonster">抽取</button>
        </div>
        <div class="value">{{ result.monster || '???' }}</div>
      </div>

      <div class="card">
        <div class="card-header">
          <h2>装备限制 (Equipment)</h2>
          <div class="card-controls">
            <input type="number" v-model="config.equipmentCount" min="0" max="5" title="数量">
            <button class="mini-btn" @click="generateEquipment">抽取</button>
          </div>
        </div>
        <ul v-if="result.equipment.length">
          <li v-for="(item, index) in result.equipment" :key="index">{{ item }}</li>
        </ul>
        <div v-else class="placeholder">无限制</div>
      </div>

      <div class="card">
        <div class="card-header">
          <h2>行为限制 (Behavior)</h2>
          <div class="card-controls">
            <input type="number" v-model="config.behaviorCount" min="0" max="5" title="数量">
            <button class="mini-btn" @click="generateBehavior">抽取</button>
          </div>
        </div>
        <ul v-if="result.behavior.length">
          <li v-for="(item, index) in result.behavior" :key="index">{{ item }}</li>
        </ul>
        <div v-else class="placeholder">无限制</div>
      </div>

      <div class="card">
        <div class="card-header">
          <h2>特殊限制 (Special)</h2>
          <div class="card-controls">
            <input type="number" v-model="config.specialCount" min="0" max="5" title="数量">
            <button class="mini-btn" @click="generateSpecial">抽取</button>
          </div>
        </div>
        <ul v-if="result.special.length">
          <li v-for="(item, index) in result.special" :key="index">{{ item }}</li>
        </ul>
        <div v-else class="placeholder">无限制</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

header h1 {
  color: #f0b90b;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  margin-bottom: 0.5rem;
}

header p {
  color: #aaa;
  margin-bottom: 2rem;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-bottom: 2rem;
  flex-wrap: wrap;
}

.control-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

input[type="number"] {
  background: #333;
  border: 1px solid #555;
  color: #fff;
  padding: 0.5rem;
  border-radius: 4px;
  width: 60px;
  text-align: center;
}

.generate-btn {
  background: linear-gradient(45deg, #8b0000, #cd5c5c);
  color: white;
  border: none;
  padding: 1rem 3rem;
  font-size: 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s;
  box-shadow: 0 4px 15px rgba(0,0,0,0.3);
  font-weight: bold;
  margin-bottom: 3rem;
}

.generate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0,0,0,0.4);
}

.generate-btn:active {
  transform: translateY(1px);
}

.results {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
}

.card {
  background: #2a2a2a;
  border: 1px solid #444;
  border-radius: 8px;
  padding: 1.5rem;
  box-shadow: 0 4px 6px rgba(0,0,0,0.2);
}

.weapon-card, .monster-card {
  border-color: #f0b90b;
  background: #2a2a2a;
}

.card h2 {
  color: #f0b90b;
  font-size: 1.1rem;
  margin: 0;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #444;
  padding-bottom: 0.5rem;
  margin-bottom: 1rem;
}

.card-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.mini-btn {
  background: #444;
  color: #fff;
  border: 1px solid #666;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.8rem;
  transition: background 0.2s;
}

.mini-btn:hover {
  background: #555;
}

.placeholder {
  color: #666;
  font-style: italic;
}

.value {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
  text-align: left;
}

li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #333;
}

li:last-child {
  border-bottom: none;
}
</style>
