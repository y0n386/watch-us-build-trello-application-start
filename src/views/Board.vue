<template>
  <div class="board">
    <div class="flex flex-row items-start">
      <div class="column" v-for="(column,columnIndex) in board.columns" :key="columnIndex"
           @drop="moveTask($event,column.tasks)"
           @dragover.prevent>
        <div>{{column.name}}</div>
        <div class="task" v-for="(task,taskIndex) in column.tasks"
             :key="taskIndex" @click="goTaskOpen(task)"
             draggable="true"
             @dragstart="pickUpTask($event,taskIndex,columnIndex)">
          <div class="text-bold">{{task.name}}</div>
          <!--          <div>{{task.description}}</div>-->

        </div>
        <input class="w-full block p-2 bg-transparent"
               placeholder=" name + Enter "
               @keyup.enter="addTask($event, column.tasks)">
      </div>

    </div>
    <div class="task-bg" v-if="isTaskOpen" @click.self="closeModal()">
      <router-view></router-view>
    </div>
  </div>
</template>

<script>
    import {mapState} from 'vuex'

    export default {
        computed: {
            ...mapState(['board']),
            isTaskOpen() {
                return this.$route.name === 'task'
            }
        },
        methods: {
            goTaskOpen(task) {
                debugger
                this.$router.push({name: 'task', params: {id: task.id}})
            },
            closeModal() {
                this.$router.push({name: 'board'})
            },
            addTask(e, tasks) {
                this.$store.commit('CREATE_TASK', {tasks, name: e.target.value});
            },
            pickUpTask(e, taskIndex, columnIndex) {
                e.dataTransfer.effectAllowed = 'move';
                e.dataTransfer.dropEffect = 'move';

                e.dataTransfer.setData('task-index', taskIndex);
                e.dataTransfer.setData('from-column-index', columnIndex);
            },
            moveTask(e, toTasks) {
                debugger
                const fromColumnIndex = e.dataTransfer.getData('from-column-index');
                const fromTasks = this.board.columns[fromColumnIndex].tasks;
                const taskIndex = e.dataTransfer.getData('task-index');

                this.$store.commit('MOVE_TASK',{
                    fromTasks,
                    toTasks,
                    taskIndex
                })

            }
        }
    }
</script>

<style lang="css">
  .task {
    @apply flex items-center flex-wrap shadow mb-2 py-2 px-2 rounded bg-white text-grey-darkest no-underline;
  }

  .column {
    @apply bg-grey-light p-2 mr-4 text-left shadow rounded;
    min-width: 350px;
  }

  .board {
    @apply p-4 bg-teal-dark h-full overflow-auto;
  }

  .task-bg {
    @apply pin absolute;
    background: rgba(0, 0, 0, 0.5);
  }
</style>
