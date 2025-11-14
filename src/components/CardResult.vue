<template>
  <div class="card-result">
    <div class="card-display">
      <h2>{{ card.name }}</h2>
      <p class="fortune">{{ card.fortune }}</p>
    </div>
    <div class="result-buttons">
      <button class="btn btn-primary" @click="$emit('generate')">生成理由 📄</button>
      <button class="btn btn-secondary" @click="$emit('again')">再抽一次 🔁</button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  card: {
    type: Object,
    required: true,
  },
})

defineEmits(['generate', 'again'])
</script>

<style scoped>
.card-result {
  text-align: center;
  padding: 30px 0;
}

.card-display {
  margin-bottom: 40px;
  animation: slideIn 0.4s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

h2 {
  font-size: 36px;
  color: #6b4ce6;
  margin-bottom: 10px;
  font-weight: 800;
}

.fortune {
  font-size: 20px;
  color: #f0a500;
  font-style: italic;
  font-weight: 600;
}

.result-buttons {
  display: flex;
  gap: 15px;
}

.btn {
  flex: 1;
  padding: 12px 20px;
  font-size: 16px;
  border: 2px solid #6b4ce6;
  background: white;
  color: #6b4ce6;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
}

.btn-primary {
  background: #6b4ce6;
  color: white;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(107, 76, 230, 0.3);
}

.btn-secondary {
  background: white;
  border-color: #6b4ce6;
}

.btn-secondary:hover {
  background: rgba(107, 76, 230, 0.1);
  transform: translateY(-2px);
}
</style>
