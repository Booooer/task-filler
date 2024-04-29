<script setup>
import { ref } from 'vue'
import { Plus } from '@element-plus/icons-vue'

defineProps({
  formTitle: String,
})

const currentTask = ref('')

const tasks = []
</script>

<template>
  <h1>{{ formTitle }}</h1>

  <div class="log-form-task">
    <div v-for="task in tasks">
      <div class="task-item">
        <p>{{ task }}</p>
      </div>
    </div>
  </div>
  <div class="add-task-input">
    <el-input
        v-model="currentTask"
        style="width: 340px"
        placeholder="Что делал сегодня?)"
    />
    <el-button :icon="Plus" />
  </div>
</template>

<style scoped>
.log-form-task {
  border: 2px solid #747bff;
  border-radius: 15px;
  text-align: left;
  padding: 10px 30px;
  max-height: 200px;
  overflow: auto;
}

.task-item {
  padding: 5px;
}

.task-item:hover {
  background: #747bff;
  transition: all .5s linear;
  cursor: copy;
}

.add-task-input{
  margin-top: 15px;
}
</style>
