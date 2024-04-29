<script setup>
import {ref} from 'vue'
import {Plus} from '@element-plus/icons-vue'
import {ElMessage} from 'element-plus'

const allTasks = ref()
allTasks.value = JSON.parse(localStorage.getItem('tasks')) ?? []
const clearStorage = () => {
  localStorage.clear()
}

const date = new Date(); // 3 января 2014 года
const getWeekDay = () => {
  let days = ['Воскресенье', 'Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота'];

  return days[date.getDay()];
}

const getReportInfo = () => {
  if (tasks.value.length === 0) {
    ElMessage.error('Ты не описал ни одной задачи, лентяй!')
  }
  let report = getWeekDay() + ':'

  tasks.value.forEach((task) => {
    report += '\n' + ' - ' + task
  })

  // allTasks.value.push(tasks)
  // localStorage.setItem('tasks',JSON.stringify(allTasks))
  navigator.clipboard.writeText(report)
}

const validateTask = (task) => {
  if (task.length < 9 || task.length > 64) {
    return false
  }
  return task.indexOf('Тест');
}
const updateStorage = (task) => {
  if (!validateTask(task)) {
    ElMessage.error('Ты уверен, что написал всё корректно?')
    return
  }
  tasks.value.push(task)
}

const copyTask = (task) => {
  navigator.clipboard.writeText(task)
}

let tasks = ref([])
const currentTask = ref('')
</script>

<template>
  <h1 v-if="allTasks">Лог задач</h1>
  <div class="log-form-task" v-if="allTasks">
    <div
        v-for="logTask in allTasks"
    >
      <div
          class="task-item"
          @click="copyTask(logTask)"
      >
        <p>{{ logTask }}</p>
      </div>
    </div>
  </div>

  <h1>Текущий список</h1>
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
    <el-button
        :icon="Plus"
        @click="updateStorage(currentTask)"
    />
  </div>
  <el-button
      style="margin-top: 15px"
      size="default"
      @click="getReportInfo()"
  >
    Сформировать отчёт
  </el-button>
</template>


<style scoped>
.log-form-task {
  border: 2px solid #747bff;
  border-radius: 15px;
  text-align: left;
  padding: 10px 30px;
  max-height: 200px;
  min-height: 100px;
  overflow: auto;
}

.task-item {
  padding: 1px 2px;
}

.task-item:hover {
  background: #747bff;
  transition: all .5s linear;
  cursor: copy;
}

.add-task-input {
  margin-top: 15px;
}
</style>
