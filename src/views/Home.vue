<template>
  <div v-if="showAddTask">
    <AddTask @add-task="addTask" />
  </div>
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>
<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";
export default {
  name: "Home",
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  props: {
    showAddTask: Boolean,
  },
  methods: {
    async addTask(task) {
      const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(task),
      });

      const data = await res.json();
      this.tasks = [...this.tasks, data];
    },
    async deleteTask(id) {
      if (confirm("Are you sure?")) {
        const res = await fetch(`api/tasks/${id}`, { method: "DELETE" });

        res.status === 200
          ? (this.tasks = this.tasks.filter((t) => t.id !== id))
          : alert("Error deleting task.");
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updatedTask = {
        ...taskToToggle,
        reminder: !taskToToggle.reminder,
      };

      const res = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(updatedTask),
      });

      const data = await res.json();

      res.status === 200
        ? (this.tasks = this.tasks.map((t) =>
            t.id === id ? { ...t, reminder: data.reminder } : t
          ))
        : alert("Error toggling reminder");
    },
    async fetchTasks() {
      const res = await fetch("api/tasks");
      const data = await res.json();
      return data;
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = await res.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
