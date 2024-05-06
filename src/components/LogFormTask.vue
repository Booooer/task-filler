<script setup>
import {ref} from 'vue'
import {Plus} from '@element-plus/icons-vue'
import {Delete} from "@element-plus/icons-vue"
import {ElMessage} from 'element-plus'

const allTasks = ref([])
allTasks.value = JSON.parse(localStorage.getItem('tasks')) ?? []
const clearStorage = () => {
  localStorage.clear()
  allTasks.value = []
}

const date = new Date()
const getWeekDay = () => {
  let days = ['Воскресенье', 'Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота']

  return days[date.getDay()]
}

const getReportInfo = () => {
  if (tasks.value.length === 0) {
    ElMessage.error('Ты не описал ни одной задачи, лентяй!')
  }
  let report = getWeekDay() + ':'

  tasks.value.forEach((task) => {
    report += '\n' + ' - ' + task
  })

  localStorage.setItem('tasks', JSON.stringify(allTasks))
  navigator.clipboard.writeText(report)
  setLocalStorage()
}

const setLocalStorage = () => {
  let prepareTasks = allTasks.value

  if (prepareTasks.length === 0) {
    allTasks.value = tasks.value
    return localStorage.setItem('tasks', JSON.stringify(tasks.value))
  }

  tasks.value.forEach((task) => {
    if (!prepareTasks.includes(task) || prepareTasks.length === 0) {
      prepareTasks.push(task)
    }
  })

  localStorage.setItem('tasks', JSON.stringify(prepareTasks))
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
  currentTask.value = ''
}

const copyTask = (task) => {
  navigator.clipboard.writeText(task)
}

const removeItemTask = (index) => {
  tasks.value.splice(index, 1)
}

const removeLogItem = (index) => {
  allTasks.value.splice(index,1)
  localStorage.setItem('tasks',JSON.stringify(allTasks.value))
}

const tasks = ref([])
const currentTask = ref('')

</script>

<template>
  <h1 v-if="allTasks">Лог задач</h1>
  <div class="log-form-task" v-if="allTasks">
    <div
        v-for="(logTask,index) in allTasks"
    >
      <div
          class="task-item"
          @click="copyTask(logTask)"
      >
        <p>{{ logTask }}</p>
        <div class="rm-button">
          <el-icon
              @click="removeLogItem(index)">
            <Delete/>
          </el-icon>
        </div>
      </div>
    </div>
  </div>

  <h1>Текущий список</h1>
  <div class="log-form-task">
    <div v-for="(task, index) in tasks">
      <div class="task-item">
        <p>{{ task }}</p>
        <div class="rm-button">
          <el-icon
              @click="removeItemTask(index)">
            <Delete/>
          </el-icon>
        </div>
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
  <el-button
      style="margin-top: 15px"
      size="default"
      @click="clearStorage()"
  >
    Очистить лог
  </el-button>
</template>


<style scoped>
.log-form-task {
  position: relative;
  border: 2px solid #747bff;
  border-radius: 15px;
  text-align: left;
  width: 50vw;
  min-width: 400px;
  min-height: 100px;
  overflow: auto;
}

.task-item {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  align-content: center;
}

.task-item > div {
  padding-right: 10px;
  transition: all .5s linear;
  cursor: pointer;
}

.task-item > p {
  padding-left: 10px;
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
