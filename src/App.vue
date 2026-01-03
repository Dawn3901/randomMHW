<script setup>
import { ref, reactive, watch, onMounted, computed } from 'vue';
import { options as defaultOptions } from './data/options.js';

const lang = ref('zh'); // 'zh' or 'en'

const translations = {
  zh: {
    title: 'MHW 随机挑战生成器',
    subtitle: '愿指引明路的苍蓝星为你闪耀！',
    generateAll: '一键生成所有',
    managePools: '管理随机池',
    weapon: '武器',
    monster: '目标怪物',
    equipment: '装备限制',
    behavior: '行为限制',
    special: '特殊限制',
    draw: '抽取',
    noLimit: '无限制',
    reset: '重置为默认',
    done: '完成',
    addPlaceholderZh: '中文描述...',
    addPlaceholderEn: '英文描述...',
    noOptions: '无可用选项',
    unknown: '???',
    clear: '清空'
  },
  en: {
    title: 'MHW Random Challenge Generator',
    subtitle: 'May the Sapphire Star light your way!',
    generateAll: 'Generate All',
    managePools: 'Manage Pools',
    weapon: 'Weapon',
    monster: 'Target Monster',
    equipment: 'Equipment Restrictions',
    behavior: 'Behavior Restrictions',
    special: 'Special Restrictions',
    draw: 'Draw',
    noLimit: 'No Restrictions',
    reset: 'Reset to Default',
    done: 'Done',
    addPlaceholderZh: 'Chinese Desc...',
    addPlaceholderEn: 'English Desc...',
    noOptions: 'No Options',
    unknown: '???',
    clear: 'Clear'
  }
};

const t = computed(() => translations[lang.value]);

const toggleLang = () => {
  lang.value = lang.value === 'zh' ? 'en' : 'zh';
};

const result = reactive({
  weapon: null,
  monster: null,
  equipment: [],
  behavior: [],
  special: []
});

const config = reactive({
  equipmentCount: 1,
  behaviorCount: 1,
  specialCount: 1
});

// Reactive pool data
const pool = reactive({
  weapons: [],
  monsters: [],
  equipmentRestrictions: [],
  behaviorRestrictions: [],
  specialRestrictions: []
});

const isEditing = ref(false);
const newItem = reactive({
  weapons: { zh: '', en: '' },
  monsters: { zh: '', en: '' },
  equipmentRestrictions: { zh: '', en: '' },
  behaviorRestrictions: { zh: '', en: '' },
  specialRestrictions: { zh: '', en: '' }
});

const categoryKeys = ['weapons', 'monsters', 'equipmentRestrictions', 'behaviorRestrictions', 'specialRestrictions'];

// Initialize pool from localStorage or default
onMounted(() => {
  const savedPool = localStorage.getItem('mhw_random_pool_v2');
  if (savedPool) {
    Object.assign(pool, JSON.parse(savedPool));
  } else {
    resetPools();
  }
});

// Save to localStorage whenever pool changes
watch(pool, (newPool) => {
  localStorage.setItem('mhw_random_pool_v2', JSON.stringify(newPool));
}, { deep: true });

const resetPools = () => {
  // Deep copy default options
  // Ensure we copy the objects correctly
  pool.weapons = defaultOptions.weapons.map(i => ({...i}));
  pool.monsters = defaultOptions.monsters.map(i => ({...i}));
  pool.equipmentRestrictions = defaultOptions.equipmentRestrictions.map(i => ({...i}));
  pool.behaviorRestrictions = defaultOptions.behaviorRestrictions.map(i => ({...i}));
  pool.specialRestrictions = defaultOptions.specialRestrictions.map(i => ({...i}));
};

const addItem = (category) => {
  if (newItem[category].zh.trim() || newItem[category].en.trim()) {
    pool[category].push({
      zh: newItem[category].zh.trim() || newItem[category].en.trim(),
      en: newItem[category].en.trim() || newItem[category].zh.trim()
    });
    newItem[category].zh = '';
    newItem[category].en = '';
  }
};

const removeItem = (category, index) => {
  pool[category].splice(index, 1);
};

const getRandomItem = (arr) => {
  if (!arr || arr.length === 0) return null;
  return arr[Math.floor(Math.random() * arr.length)];
};

const getRandomItems = (arr, count) => {
  if (!arr || arr.length === 0) return [];
  const shuffled = [...arr].sort(() => 0.5 - Math.random());
  return shuffled.slice(0, count);
};

const addUniqueItems = (targetList, sourcePool, count) => {
  if (!sourcePool || sourcePool.length === 0) return;
  
  // Find items in sourcePool that are not in targetList
  const available = sourcePool.filter(sourceItem => 
    !targetList.some(targetItem => 
      targetItem.zh === sourceItem.zh && targetItem.en === sourceItem.en
    )
  );
  
  if (available.length === 0) return;
  
  const shuffled = [...available].sort(() => 0.5 - Math.random());
  const selected = shuffled.slice(0, count);
  
  targetList.push(...selected);
};

const clearItems = (list) => {
  list.splice(0, list.length);
};

const generateWeapon = () => {
  result.weapon = getRandomItem(pool.weapons);
};

const generateMonster = () => {
  result.monster = getRandomItem(pool.monsters);
};

const addEquipment = (count) => {
  addUniqueItems(result.equipment, pool.equipmentRestrictions, count);
};

const clearEquipment = () => {
  clearItems(result.equipment);
};

const addBehavior = (count) => {
  addUniqueItems(result.behavior, pool.behaviorRestrictions, count);
};

const clearBehavior = () => {
  clearItems(result.behavior);
};

const addSpecial = (count) => {
  addUniqueItems(result.special, pool.specialRestrictions, count);
};

const clearSpecial = () => {
  clearItems(result.special);
};

const generateChallenge = () => {
  generateWeapon();
  generateMonster();
  
  clearEquipment();
  addEquipment(1);
  
  clearBehavior();
  addBehavior(1);
  
  clearSpecial();
  addSpecial(1);
};

const displayItem = (item) => {
  if (!item) return t.value.unknown;
  return item[lang.value] || item.zh || item.en || t.value.unknown;
};
</script>

<template>
  <div class="container">
    <div class="lang-switch">
      <button @click="toggleLang">{{ lang === 'zh' ? 'English' : '中文' }}</button>
    </div>
    <header>
      <h1>{{ t.title }}</h1>
      <p>{{ t.subtitle }}</p>
    </header>

    <div class="main-actions">
      <button class="generate-btn" @click="generateChallenge">{{ t.generateAll }}</button>
      <button class="manage-btn" @click="isEditing = true">{{ t.managePools }}</button>
    </div>

    <div class="results">
      <div class="card weapon-card">
        <div class="card-header">
          <h2>{{ t.weapon }}</h2>
          <button class="mini-btn" @click="generateWeapon">{{ t.draw }}</button>
        </div>
        <div class="value">{{ displayItem(result.weapon) }}</div>
      </div>

      <div class="card monster-card">
        <div class="card-header">
          <h2>{{ t.monster }}</h2>
          <button class="mini-btn" @click="generateMonster">{{ t.draw }}</button>
        </div>
        <div class="value">{{ displayItem(result.monster) }}</div>
      </div>

      <div class="card">
        <div class="card-header">
          <h2>{{ t.equipment }}</h2>
          <div class="card-controls">
            <button class="mini-btn" @click="addEquipment(1)">+1</button>
            <button class="mini-btn" @click="addEquipment(3)">+3</button>
            <button class="mini-btn clear-btn" @click="clearEquipment">{{ t.clear }}</button>
          </div>
        </div>
        <ul v-if="result.equipment.length">
          <li v-for="(item, index) in result.equipment" :key="index">{{ displayItem(item) }}</li>
        </ul>
        <div v-else class="placeholder">{{ t.noLimit }}</div>
      </div>

      <div class="card">
        <div class="card-header">
          <h2>{{ t.behavior }}</h2>
          <div class="card-controls">
            <button class="mini-btn" @click="addBehavior(1)">+1</button>
            <button class="mini-btn" @click="addBehavior(3)">+3</button>
            <button class="mini-btn clear-btn" @click="clearBehavior">{{ t.clear }}</button>
          </div>
        </div>
        <ul v-if="result.behavior.length">
          <li v-for="(item, index) in result.behavior" :key="index">{{ displayItem(item) }}</li>
        </ul>
        <div v-else class="placeholder">{{ t.noLimit }}</div>
      </div>

      <div class="card">
        <div class="card-header">
          <h2>{{ t.special }}</h2>
          <div class="card-controls">
            <button class="mini-btn" @click="addSpecial(1)">+1</button>
            <button class="mini-btn" @click="addSpecial(3)">+3</button>
            <button class="mini-btn clear-btn" @click="clearSpecial">{{ t.clear }}</button>
          </div>
        </div>
        <ul v-if="result.special.length">
          <li v-for="(item, index) in result.special" :key="index">{{ displayItem(item) }}</li>
        </ul>
        <div v-else class="placeholder">{{ t.noLimit }}</div>
      </div>
    </div>

    <!-- Edit Modal -->
    <div class="modal-overlay" v-if="isEditing" @click.self="isEditing = false">
      <div class="modal-content">
        <div class="modal-header">
          <h2>{{ t.managePools }}</h2>
          <button class="close-btn" @click="isEditing = false">×</button>
        </div>
        
        <div class="modal-body">
          <div class="pool-section" v-for="key in categoryKeys" :key="key">
            <h3>{{ t[key.replace('Restrictions', '')] || t[key] }}</h3>
            <div class="add-item">
              <input v-model="newItem[key].zh" :placeholder="t.addPlaceholderZh">
              <input v-model="newItem[key].en" :placeholder="t.addPlaceholderEn" @keyup.enter="addItem(key)">
              <button @click="addItem(key)">+</button>
            </div>
            <ul class="pool-list">
              <li v-for="(item, index) in pool[key]" :key="index">
                <span>{{ item.zh }} / {{ item.en }}</span>
                <button class="remove-btn" @click="removeItem(key, index)">×</button>
              </li>
            </ul>
          </div>
        </div>

        <div class="modal-footer">
          <button class="reset-btn" @click="resetPools">{{ t.reset }}</button>
          <button class="done-btn" @click="isEditing = false">{{ t.done }}</button>
        </div>
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
  position: relative;
}

.lang-switch {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.lang-switch button {
  background: transparent;
  border: 1px solid #666;
  color: #aaa;
  padding: 0.3rem 0.8rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.lang-switch button:hover {
  color: #fff;
  border-color: #fff;
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

.clear-btn {
  background: #8b0000;
  border-color: #a00000;
}

.clear-btn:hover {
  background: #a00000;
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

.main-actions {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 3rem;
  flex-wrap: wrap;
}

.manage-btn {
  background: #444;
  color: white;
  border: 1px solid #666;
  padding: 1rem 3rem;
  font-size: 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
  font-weight: bold;
}

.manage-btn:hover {
  background: #555;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: #2a2a2a;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  border: 1px solid #444;
  box-shadow: 0 10px 25px rgba(0,0,0,0.5);
}

.modal-header {
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #444;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header h2 {
  margin: 0;
  color: #f0b90b;
}

.close-btn {
  background: none;
  border: none;
  color: #aaa;
  font-size: 2rem;
  cursor: pointer;
  padding: 0;
  line-height: 1;
}

.close-btn:hover {
  color: #fff;
}

.modal-body {
  padding: 1.5rem;
  overflow-y: auto;
  flex: 1;
}

.pool-section {
  margin-bottom: 2rem;
}

.pool-section h3 {
  color: #ddd;
  border-bottom: 1px solid #444;
  padding-bottom: 0.5rem;
  margin-bottom: 1rem;
}

.add-item {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.add-item input {
  flex: 1;
  background: #333;
  border: 1px solid #555;
  color: #fff;
  padding: 0.5rem;
  border-radius: 4px;
}

.add-item button {
  background: #4caf50;
  color: white;
  border: none;
  padding: 0 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1.2rem;
}

.pool-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 0.5rem;
}

.pool-list li {
  background: #333;
  padding: 0.5rem;
  border-radius: 4px;
  border: none;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.9rem;
}

.remove-btn {
  background: none;
  border: none;
  color: #ff6b6b;
  cursor: pointer;
  font-size: 1.2rem;
  padding: 0 0.5rem;
}

.remove-btn:hover {
  color: #ff4c4c;
}

.modal-footer {
  padding: 1rem 1.5rem;
  border-top: 1px solid #444;
  display: flex;
  justify-content: space-between;
}

.reset-btn {
  background: #8b0000;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
}

.done-btn {
  background: #4caf50;
  color: white;
  border: none;
  padding: 0.5rem 1.5rem;
  border-radius: 4px;
  cursor: pointer;
}
</style>
