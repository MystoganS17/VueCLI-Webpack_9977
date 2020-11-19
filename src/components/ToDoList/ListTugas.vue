<template>
  <v-main class="list">
    <h3 class="text-h3 font-weight-medium mb-5">To Do List UGD</h3>
    <v-card>
      <v-card-title>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          hide-details
        ></v-text-field>
        <v-spacer></v-spacer>
        <v-btn color="purple" dark @click="dialogSaveDelete = true"> 
            To Do Selesai 
        </v-btn>
        <v-spacer></v-spacer>
        <v-btn color="success" dark @click="dialog = true"> 
            Tambah 
        </v-btn>
        </v-card-title>
        <v-data-table
            :headers="headers"
            :items="todos"
            :search="search"  
        >
        <template v-slot:[`item.priority`]="{ item }">
          <v-chip :color="getColor(item.priority)" label outlined dark>
            {{ item.priority }}
          </v-chip>
        </template>

        <template v-slot:[`item.actions`]="{ item }">
          <v-btn
            small class="mr-2"
            @click="dialogEdit = true;
            itemContent = item;
            fillField();"
          >
            {{ index }}edit
          </v-btn>
          <v-btn
            small 
            @click="dialogDelete = true;
            indexItem = todos.indexOf(item);"
          >
            delete
          </v-btn>
        </template>

        <v-textarea
            v-model="formTodo.note"
            label="Note"
            required
        ></v-textarea>

        <template v-slot:[`item.checkboxes`]="{ item }">
            <v-checkbox v-model="deleteChecked" :value="item"></v-checkbox>
        </template>

      </v-data-table>
    </v-card>

    <span v-if="deleteChecked.length!=0">
        <v-card align="justify" class="my-5">
            <div class="container">
                Delete Multiple :
                <ul v-for="(item,index) in deleteChecked" :key="index">
                    <li>{{ item.task }}</li>
                </ul>
                <v-btn color="red black--text" @click="multipleDelete">
                    HAPUS SEMUA
                </v-btn>
            </div>
        </v-card>
    </span>


    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Form Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field
              v-model="formTodo.task"
              label="Task"
              required
            ></v-text-field>
            <v-select
              v-model="formTodo.priority"
              :items="['Penting', 'Biasa', 'Tidak penting']"
              label="Priority"
              required
            ></v-select>
            <v-textarea
              v-model="formTodo.note"
              label="Note"
              required
            ></v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancel"> Cancel </v-btn>
          <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogSaveDelete" max-width="600px">
      <v-data-table :headers="headersSelesai" :items="todosSelesai" :search="search">
            <template v-slot:[`item.priority`]="{ item }">
              <v-chip v-if="item.priority == 'Penting'"
                  class="ma-2"
                  color="red"
                  label
                  outlined>
                  {{ item.priority }}
              </v-chip>
              <v-chip v-else-if="item.priority == 'Biasa'"
                  class="ma-2"
                  color="blue"
                  label
                  outlined>
                  {{ item.priority }}
              </v-chip>
              <v-chip v-else
                  class="ma-2"
                  color="green"
                  label
                  outlined>
                  {{ item.priority }}
              </v-chip>
          </template>
      </v-data-table>
      
      <v-card>
          <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="dialogSaveDelete = false">
                  Close
              </v-btn>
          </v-card-actions>
      </v-card>
      
    </v-dialog>

    <v-dialog v-model="dialogEdit" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Form Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field
              v-model="formTodoEdit.task"
              label="Task"
              required
              :value="itemContent.task"
            ></v-text-field>
            <v-select
              v-model="formTodoEdit.priority"
              :items="['Penting', 'Biasa', 'Tidak penting']"
              label="Priority"
              required
            ></v-select>
            <v-textarea v-model="formTodoEdit.note" label="Note" required>{{
              itemContent.note
            }}</v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancelEdit"> Cancel </v-btn>
          <v-btn color="blue darken-1" text @click="saveEdit"> Save </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogDelete" persistent max-width="400px">
      <v-card>
        <v-card-title>
          <span class="headline"><b>Yakin menghapus?</b></span>
        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="cancelDelete">
            TIDAK
          </v-btn>
          <v-btn color="red darken-1" text @click="saveDelete">
            YA
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-main>
</template>
<script>
export default {
  name: "List",
  data() {
    return {
      search: null,
      dialog: false,
      dialogDelete: false,
      dialogEdit: false,
      dialogSaveDelete: false,
      todosSelesai: [],
      itemContent: [],
      deleteChecked: [],
      headers: [
        {
          text: "Task",
          align: "start",
          sortable: true,
          value: "task",
        },
        { text: "Priority", value: "priority" },
        { text: "Note", value: "note" },
        { text: "Actions", value: "actions" },
        { text: "Multiple Delete", value: "checkboxes" },
      ],
      headersSelesai: [
        {
            text: "Task",
            align: "start",
            sortable: true,
            value: "task",
        },
        { text: "Priority", value: "priority" },
        { text: "Note", value: "note" },
      ],
      todos: [
        {
          task: "bernafas",
          priority: "Penting",
          note: "huffttt",
        },
        {
          task: "nongkrong",
          priority: "Tidak penting",
          note: "bersama tman2",
        },
        {
          task: "masak",
          priority: "Biasa",
          note: "masak air 500ml",
        },
      ],
      formTodo: {
        task: null,
        priority: null,
        note: null,
      },
      formTodoEdit: {
        task: null,
        priority: null,
        note: null,
      },
    };
  },
  methods: {
    fillField() {
      this.formTodoEdit.task = this.itemContent.task;
      this.formTodoEdit.priority = this.itemContent.priority;
      this.formTodoEdit.note = this.itemContent.note;
    },
    save() {
      this.todos.push(this.formTodo);
      this.resetForm();
      this.dialog = false;
    },
    saveEdit() {
      this.itemContent.task = this.formTodoEdit.task;
      this.itemContent.priority = this.formTodoEdit.priority;
      this.itemContent.note = this.formTodoEdit.note;
      this.resetForm();
      this.dialogEdit = false;
    },
    cancel() {
      this.resetForm();
      this.dialog = false;
    },
    cancelEdit() {
      this.resetForm();
      this.dialogEdit = false;
    },
    cancelDelete() {
      this.dialogDelete = false;
    },
    deleteItem(item) {
      this.index = this.todos.indexOf(item);
    },
    resetForm() {
      this.formTodo = {
        task: null,
        priority: null,
        note: null,
      };
      this.formTodoEdit = {
        task: null,
        priority: null,
        note: null,
      };
    },
    saveDelete() {
        this.todosSelesai.push(this.todos[this.index]),
        this.todos.splice(this.index, 1);
        this.dialogDelete=false;
    },
    getColor(priority) {
      if (priority == "Penting") return "red";
      else if (priority == "Tidak penting") return "green";
      else if (priority == "Biasa") return "blue";
    },
    multipleDelete()
    {
        for(this.i = 0; this.i < this.deleteChecked.length; this.i++)
        {
            this.index = this.todos.indexOf(this.deleteChecked[this.i]);
            this.todosSelesai.push(this.todos[this.index]);
            this.todos.splice(this.index, 1);
        }
        this.deleteChecked = [];
    },
  },
};
</script>