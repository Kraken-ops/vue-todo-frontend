<template>
  <div class="todo pa-6">
    <v-row>
      <h1 class="ml-6">Todo page</h1>
      <v-spacer />
      <!-- <input
          type="text"
          v-model="search"
          placeholder="Search tasks..."
          @keyup="getTask"
        />
        <br />
        
        <v-spacer></v-spacer> -->

      <div style="margin-bottom: 17px;
    margin-right: 19px;">
        <span v-if="searchBtn == false" class="d-flex">
          <v-btn
            class="ma-2 vh-90 "
            text
            icon
            color="blue darken-2"
            @click="searchBtn = true"
          >
            <v-icon>mdi-magnify</v-icon>
          </v-btn>
        </span>
        <span v-if="searchBtn == true">
          <!-- <v-row> -->
            <!-- <v-col> -->
              <span class="d-flex mt-2 vh-90  ">
              <v-text-field
                v-model="search"
                dense
                label="Search"
                @change="viewTask"
                autofocus
              ></v-text-field>
              <v-btn
                class="mr-2"
                text
                icon
                color="blue darken-2"
                @click="searchBtn = false"
              >
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </span>
            <!-- </v-col>
            <v-col>
              
            </v-col> -->
          <!-- </v-row> -->
        </span>
      </div>
    </v-row>

    <v-text-field
      class="pa-3"
      required
      outlined
      clearable
      label="Add Todo"
      v-model="value"
      hide-details="auto"
      :rules="textRules"
      :append-icon="value ? 'mdi-plus' : ''"
      @click:append="addTask"
      @keyup.enter="addTask"
      autofocus
    ></v-text-field>
    <div v-for="task in filterTask" :key="task.id">
      <v-list-item
        @click="doneTask(task)"
        :class="{ 'blue lighten-4': task.done }"
      >
        <template v-slot:default>
          <v-list-item-action>
            <!-- <v-checkbox :input-value="task.done" color="primary"></v-checkbox> -->
            <v-checkbox v-model="task.done" color="primary"></v-checkbox>
          </v-list-item-action>

          <v-list-item-content>
            <v-list-item-title
              :class="{ 'text-decoration-line-through': task.done }"
            >
              {{ task.text }}
            </v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <v-btn @click.stop="deleteTask(task._id)" icon>
              <v-icon color="blue lighten-1">mdi-delete-circle-outline</v-icon>
            </v-btn>
          </v-list-item-action>
        </template>
      </v-list-item>
    </div>

    <div class="text-center">
      <!-- <v-btn
      dark
      color="red darken-2"
      @click="snackbar = true"
    >
      Open Snackbar
    </v-btn> -->

      <v-snackbar
        v-model="snackbar"
        :multi-line="multiLine"
        :color="snackbarColor"
      >
        {{ snackbarText }}

        <template v-slot:action="{ attrs }">
          <v-btn color="red" text v-bind="attrs" @click="snackbar = false">
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const { v4: uuidv4 } = require("uuid");
export default {
  name: "todo",
  data() {
    return {
      searchBtn: false,
      multiLine: true,
      snackbar: false,
      snackbarText: "",
      snackbarColor: "",
      arr: [],
      value: "",
      textRules: [(v) => !!v || "Please enter something"],
      // tasks: [
      //   { id: 0, text: "Wake up", done: false },
      //   { id: 1, text: "Get Eggs", done: false },
      //   { id: 2, text: "Get Bread", done: false },
      //   { id: 3, text: "Make Breakfast", done: false },
      // ],
      tasks: [],
      filterTask: [],
      search: "",
    };
  },
  mounted() {
    this.getTasks();
  },
  computed: {
    textfieldShow() {
      if (this.value.length() > 0) {
        return addTask();
      }
      // return (this.value.trim().length() > 0 ? addTask() : disabled )
    },
    viewTask() {
      // if (this.search.trim() === "") {
      //   this.filterTask = this.tasks;
      // } else {
      //   this.filterTask = this.tasks.filter((item, index) => {
      //     return item.text === this.search;
      //   });
      // }
      this.filterTask = this.tasks;

      if (this.search.trim()) {
        this.filterTask = this.filterTask.filter((task) =>
          task.text.includes(this.search.trim())
        );
      }
    },
  },
  methods: {
    onKeyUp(event) {
      if (event.keyCode === 8) {
        // backspace key
        //this.search = ''
        let clearArr = this.search.split("");
        clearArr.pop();
        this.search = clearArr.join("").toString();
      }
    },
    getTasks() {
      axios
        .get("http://localhost:3000/todos")
        // .then(response => response.data)
        .then((response) => {
          this.tasks = response.data;
          this.filterTask = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
      // .then(res => {
      //   //console.log(res._id);
      //   if (this.search) {
      //     this.tasks = res.text.filter(people =>
      //       people.name.toLowerCase().includes(this.search.toLowerCase())
      //     );
      //   } else {
      //     this.people = res.results;
      //   }
      // });
    },
    addTask() {
      if (this.value.length > 0) {
        axios
          .post("http://localhost:3000/todos", {
            // id: uuidv4(),
            id: Date.now(),
            text: this.value,
            done: false,
          })
          .then((response) => {
            console.log(response.data);
            this.value = "";
            this.snackbar = true;
            this.snackbarText = "task added successfully";
            this.getTasks();
          })
          .catch((error) => {
            this.snackbarText = "oops error occured";
            console.error(error);
          });
      }
    },
    doneTask(task) {
      // console.log("id: ", id);
      // let task = this.tasks.filter((task) => task.id == id)[0];
      // task.done = !task.done;

      axios
        .put(`http://localhost:3000/todos/${task._id}/status`, {
          done: task.done,
        })
        .then((response) => {
          console.log(response.data);
          // let task = this.tasks.filter((task) => task.id == id)[0];
          // task.done = !task.done;

          if (task.done === true) {
            this.snackbar = true;
            this.snackbarText = "task done successfully";
            this.snackbarColor = "primary";
          }

          this.getTasks();
        })
        .catch((error) => {
          this.snackbarText = "oops error occured";
          console.error(error);
        });
    },
    deleteTask(_id) {
      axios
        .delete(`http://localhost:3000/todos/delete/${_id}`)
        .then((response) => {
          console.log(response.data);
          this.snackbar = true;
          this.snackbarText = "task deleted successfully";
          this.getTasks();
        })
        .catch((error) => {
          this.snackbarText = "oops error occured";
          console.error(error);
        });
    },
  },
  updateId(tasks) {
    for (index of this.tasks) {
      this.arr.push(index.id);
    }
    return Math.min.apply(null, this.arr) + 1;
  },
};
</script>
<!-- <style>
#searchFix{
  margin-right: 40%;
  padding-right: 20%;
}
</style> -->