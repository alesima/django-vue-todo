<template>
  <div class="home">
    <div class="title">Todo</div>

    <hr />

    <div class="columns">
      <div class="column is-one-quarter is-offset-two-fifths">
        <form v-on:submit.prevent="addTask">
          <h2 class="subtitle">Add task</h2>
          <div class="field">
            <label for="description" class="label">Description</label>
            <div class="control">
              <input
                type="text"
                class="input"
                name="description"
                v-model="description"
              />
            </div>
          </div>

          <div class="field">
            <label for="status" class="label">Status</label>
            <div class="control">
              <div class="select">
                <select name="status" v-model="status">
                  <option value="todo">To do</option>
                  <option value="done">Done</option>
                </select>
              </div>
            </div>
          </div>

          <div class="field">
            <div class="control">
              <button class="button is-link">Submit</button>
            </div>
          </div>
        </form>
      </div>
    </div>

    <div class="columns">
      <div class="column is-5 is-offset-1">
        <h2 class="subtitle">To do</h2>
        <div class="todo">
          <div class="card" v-for="task in tasksTodo" :key="task.id">
            <div class="card-content">{{ task.description }}</div>
            <footer class="card-footer">
              <a
                class="card-footer-item"
                v-on:click="setStatus(task.id, 'done')"
                >Done</a
              >
            </footer>
          </div>
        </div>
      </div>
      <div class="column is-5">
        <h2 class="subtitle">Done</h2>
        <div class="todo" v-for="task in tasksDone" :key="task.id">
          <div class="card">
            <div class="card-content">{{ task.description }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const axiosClient = axios.create({
  baseURL: "http://127.0.0.1:8000/",
  auth: {
    username: "admin",
    password: "123456",
  },
  headers: {
    "Content-Type": "application/json",
  },
});

export default {
  name: "HomeView",
  data() {
    return {
      tasks: [],
      description: "",
      status: "todo",
    };
  },
  mounted() {
    this.getTasks();
  },
  methods: {
    getTasks() {
      axiosClient
        .get("tasks/")
        .then((response) => (this.tasks = response.data));
    },
    addTask() {
      if (this.description) {
        axiosClient
          .post("tasks/", {
            description: this.description,
            status: this.status,
          })
          .then((response) => {
            const newTask = {
              id: response.data.id,
              description: this.description,
              status: this.status,
            };
            this.tasks.push(newTask);
            this.resetForm();
          })
          .catch((error) => console.log(error));
      }
    },
    setStatus(taskId, status) {
      axiosClient
        .patch(`/tasks/${taskId}/`, {
          status,
        })
        .then(() => {
          this.tasks = this.tasks.map((task) => {
            if (task.id === taskId) {
              return { ...task, status };
            }
            return task;
          });
        })
        .catch((error) => {
          console.log(error);
        });
    },
    resetForm() {
      this.description = "";
      this.status = "todo";
    },
  },
  computed: {
    tasksTodo() {
      return this.tasks.filter((task) => task.status === "todo");
    },
    tasksDone() {
      return this.tasks.filter((task) => task.status === "done");
    },
  },
};
</script>

<style lang="scss">
.select,
select {
  width: 100%;
}
.card {
  margin-bottom: 20px;
}
.done {
  opacity: 0.3;
}
</style>
