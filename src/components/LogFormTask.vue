<script setup>
import {ref} from 'vue'
import {Plus} from '@element-plus/icons-vue'
import {Minus} from '@element-plus/icons-vue'
import {Delete} from "@element-plus/icons-vue"
import {ElMessage} from 'element-plus'
import {isEmpty} from "element-plus/es/utils/index";

const allTasks = ref([])
allTasks.value = JSON.parse(localStorage.getItem('tasks')) ?? []
const clearStorage = () => {
  if (!clearValue.value) {
    ElMessage.warning('Выбери тип лога, дурачок :)')
    return
  }
  localStorage.setItem(`${clearValue.value}`, null)

  if (clearValue.value === 'tasks') {
    allTasks.value = []
  } else {
    projects.value = ['Б24']
  }

  ElMessage.success('Выбранный лог был успешно очищен!')
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
  return !(task.length < 9 || task.length > 96);
}
const updateStorage = (task) => {
  if (!validateTask(task)) {
    ElMessage.error('Ты уверен, что написал всё корректно?')
    return
  }

  if (currentProject.value.length > 0) {
    tasks.value.push(currentProject.value + ' - ' + checkTaskName(task))
  } else {
    tasks.value.push(task)
    updateProjects(task)
  }

  currentTask.value = ''
  currentProject.value = ''
}

const checkTaskName = (task) => {
  if (task.indexOf('-')) {
    return task.split('-')[1] ?? task.split('-')[0]
  }

  return task
}

const copyTask = (task) => {
  navigator.clipboard.writeText(task)
}

const updateProjects = (task) => {
  let project = task.split('-')[0].trim()
  if (!projects.value.includes(project) ?? project.length > 0) {
    projects.value.push(project)
    localStorage.setItem('projects', JSON.stringify(projects.value))
  }
}

const removeItemTask = (index) => {
  tasks.value.splice(index, 1)
}
const removeLogItem = (index) => {
  allTasks.value.splice(index, 1)
  localStorage.setItem('tasks', JSON.stringify(allTasks.value))
}

const switchLogShow = (isCurrent = false) => {
  if (isCurrent) {
    showCurrentTasks.value = !showCurrentTasks.value
  } else {
    showLogTasks.value = !showLogTasks.value
  }
}


let projectsCount = new Map([])
const showLogTasks = ref(true)
const showCurrentTasks = ref(true)
const tasks = ref([])
const currentTask = ref('')
const projects = ref([])
const currentProject = ref('')
const clearValue = ref('')
const countProjectValues = ref()
projects.value = !isEmpty(JSON.parse(localStorage.getItem('projects'))) ? JSON.parse(localStorage.getItem('projects')) : []

if (!projects.value.includes('Б24')) {
  projects.value.push('Б24')
}

window.addEventListener('load', () => {
  countProjectValues.value = new Map
  // tasks.value = !isEmpty(JSON.parse(localStorage.getItem('tasks'))) ? JSON.parse(localStorage.getItem('tasks')) : []
  allTasks.value.forEach((task) => {
    let project = task.split('-')[0].trim()

    if (!projectsCount.get(project)) {
      countProjectValues.value.set(project, 1)
    } else {
      let number = countProjectValues.value.get(project)
      countProjectValues.value.set(project, ++number)
    }
  })
})

</script>

<template>
  <div v-if="allTasks.length" style="display: flex; align-items: center;justify-content: center">
    <h1 v-if="allTasks.length">Лог задач</h1>
    <el-icon
        v-model="showLogTasks"
        style="width: 50px"
        v-if="showLogTasks"
        @click="switchLogShow()">
      <Minus/>
    </el-icon>
    <el-icon
        style="width: 50px"
        v-model="showLogTasks"
        v-if="!showLogTasks"
        @click="switchLogShow()">
      <Plus/>
    </el-icon>
  </div>
  <div class="log-form-task" v-if="allTasks.length && showLogTasks">
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

  <div v-if="tasks.length" style="display: flex; align-items: center;justify-content: center">
    <h1 v-if="tasks.length">Текущий список</h1>
    <el-icon
        v-model="showCurrentTasks"
        style="width: 50px"
        v-if="showCurrentTasks"
        @click="switchLogShow(true)">
      <Minus/>
    </el-icon>
    <el-icon
        style="width: 50px"
        v-model="showCurrentTasks"
        v-if="!showCurrentTasks"
        @click="switchLogShow(true)">
      <Plus/>
    </el-icon>
  </div>
  <div v-if="tasks.length && showCurrentTasks" class="log-form-task">
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
        style="width: 440px"
        placeholder="Что делал сегодня?)"
        class="input-with-select"
    >
      <template #prepend>
        <el-select
            v-model="currentProject"
            placeholder="Проект"
            style="width: 115px"
        >
          <div
              v-for="project in projects">
            <el-option
                :label="project"
                :value="project"
            />
          </div>
        </el-select>
      </template>
      <template #append>
        <el-button
            :icon="Plus"
            @click="updateStorage(currentTask)"
        />
      </template>
    </el-input>
  </div>
  <el-button
      style="margin-top: 15px"
      size="default"
      @click="getReportInfo()"
  >
    Сформировать отчёт
  </el-button>
  <div class="clear-log-btn">
    <el-input
        style="width: 340px"
        disabled
        placeholder="Что чистим?"
    >
      <template #append>
        <el-select
            v-model="clearValue"
            placeholder="Тип лога"
            style="width: 125px"
        >
          <el-option
              label="Задачи"
              value="tasks"
          />
          <el-option
              label="Проекты"
              value="projects"
          />
        </el-select>
      </template>
      <template #prepend>
        <el-button
            size="default"
            @click="clearStorage()"
            style="width: 80px"
        >
          <el-icon>
            <Delete/>
          </el-icon>
        </el-button>
      </template>
    </el-input>
  </div>
  <div
      style="width: 100%; display: flex; justify-content: center; flex-wrap: nowrap">
    <div class="stats-report">
      <el-tag
          v-for="tag in countProjectValues"
          type="primary"
          effect="dark"
      >
        {{ tag[0] }} - {{ countProjectValues.get(tag[0]) }}
      </el-tag>
    </div>
  </div>
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
  max-height: 250px;
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

.clear-log-btn {
  margin-top: 10px;
}

.stats-report {
  margin-top: 20px;
  display: flex;
  flex-wrap: wrap;
  max-width: 250px;
  align-items: center;
  gap: 7px;
}
</style>
