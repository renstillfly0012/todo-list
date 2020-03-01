<template>
  <div class="q-pa-md">
    

     <q-toolbar class="bg-primary text-white">
      <q-btn flat round dense icon="menu" class="q-mr-sm" />
      <q-avatar>
        <img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg">
      </q-avatar>

      <q-toolbar-title>Todos: {{todos.length}}</q-toolbar-title>

      <q-btn @click="downloadPDF" flat round dense icon="cloud_download" />
      <q-btn @click="openPDF" flat round dense icon="picture_as_pdf" />
    </q-toolbar>
    <!-- <input ref="n" v-model="task" type="text"> -->
    <div class="row q-ma-md q-gutter-sm">
      <q-input class="col" ref="n" v-model="task" label="Task" />
      <!-- <button @click="save">save</button> -->
      <div class="column justify-center">
        <q-btn :disable="task === ''" @click="addTask(task)" round color="primary" icon="save" />
      </div>
    </div>
    <q-list bordered>
      <q-item clickable v-ripple v-for="(t, i) in todos" :key="i">
        <q-item-section side>
          <q-checkbox @click.native="todosSrvc.patch(t._id, { isDone: !t.isDone })" :value="t.isDone" />
        </q-item-section>
        <q-item-section v-if="i !== editingIndex">{{ t.desc }}</q-item-section>
        <q-item-section v-else>
          <input ref="draft" v-model="draft" type="text">
        </q-item-section>
        <q-item-section side >
          <div v-if="i !== editingIndex" class="row">
            <q-btn @click="edit(i)" size="sm" flat icon="edit" round />
            <q-btn v-on:click="deleteIndex = t._id"  @click.native="delDialog = true" size="sm" flat icon="delete" round color="negative" />
          </div>
          <div v-else class="row">
            <q-btn   v-on:click="editingIndex = t._id"  @click.native="editDialog = true"  :disable="t.desc === draft" size="sm" flat icon="save" color="positive" round />
            <q-btn @click="editingIndex = null" size="sm" flat icon="cancel" round color="negative" />
          </div>
        </q-item-section>
      </q-item>
    </q-list>


    <pie-chart
      :donut="true"
      :data="[['finished', finished.length], ['unfinished', unfinished.length]]"
    ></pie-chart>

    <div class="q-pa-md">
      <q-dialog v-model="delDialog" persistent>
        <q-card>
          <!-- v-if="action === Del" -->
          <q-card-section class="row items-center">
            <text-h4>
              <q-icon name="warning" class="text-yellow-8" style="font-size: 2rem;" />Are you sure you want to delete this task?
            </text-h4>
          </q-card-section>

  

          <!-- Notice v-close-popup -->
          <q-card-actions align="right">
            <q-btn flat label="Cancel" color="primary" v-close-popup />
            <q-btn flat label="Yes, Delete it"    @click.native="todosSrvc.remove(deleteIndex)" color="negative" v-close-popup />

          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>

     <div class="q-pa-md">
      <q-dialog v-model="editDialog" persistent>
        <q-card>
          <!-- v-if="action === Del" -->
          <q-card-section class="row items-center">
            <text-h4>
              <q-icon name="warning" class="text-yellow-8" style="font-size: 2rem;" />Save Changes?
            </text-h4>
          </q-card-section>

          <!-- Notice v-close-popup -->
          <q-card-actions align="right">
            <q-btn flat label="Cancel" color="primary" v-close-popup />
            <q-btn flat label="Yes, Save it"  @click.native="todosSrvc.update(editingIndex, { desc: draft })" color="positive" v-close-popup />

          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>

  </div>
</template>


<style lang="stylus" scoped></style>

<script>
import greet from "src/components/greet.vue";
import { LMap, LTileLayer, LMarker } from "vue2-leaflet";
import { Notify } from "quasar";
export default {
  components: {
    greet
  },
  beforeDestroy() {
    this.todoSrvc.destroy();
  },
  mounted() {
    this.todosSrvc = this.$dbCon.wingsService("todos");
    this.todosSrvc
      .on("dataChange", tasks => {
        this.todos = tasks;
        console.log(tasks);
      })
      .init();
  },

  data() {
    //models
    return {
      todos: [
        {
          desc: "Kaen",
          isDone: false
        },
        {
          desc: "Tulog",
          isDone: false
        }
      ],
      todoSrvc: null,
      task: null,
      editingIndex: null,
      draft: "",
      delDialog: false,
      editDialog: false,
      cancelEnabled: false,
      action: null,
      deleteIndex: null,
    };
  },
  methods: {
    //controller

    openPDF() {
      this.$pdfMake
        .createPdf({
          content: [
            {
              table: {
                body: [
                  ["", "Total"],
                  ["Finished", this.finished.length],
                  ["Unfinished", this.unfinished.length]
                ]
              }
            }
          ]
        })
        .open();
    },
    downloadPDF() {
      this.$pdfMake
        .createPdf({
          content: [
            {
              table: {
                body: [
                  [" ", "Total"],
                  ["Finished", this.finished.length],
                  ["Unfinished", this.unfinished.length]
                ]
              }
            }
          ]
        })
        .download(`batch27 - ${new Date()}`);
    },
    increase(a) {
      console.log(a)
    },
    remove(index) {
      
      this.todos.splice(index, 1);
      console.log(this.index);
      this.$q.notify({
        message: "You have successfully deleted a task",
        color: "green-7"
      });
    },
    addTask(activity) {
      // this.task = "";
      // this.todos.unshift({
      //   desc: activity,
      //   isDone: false
      // })
      this.todosSrvc.create({
        desc: activity,
        isDone: false
      })

      this.task = "";
      this.$refs.n.focus();
      this.$q.notify({
        message: "You have successfully added a task",
        color: "green-7"
      })
    },
    update(ind) {
      this.editingIndex = null;
      this.todos[ind].desc = this.draft;
      this.$q.notify({
        message: "You have successfully edited task",
        color: "green-7"
      });
    },
    edit(ii) {
      this.editingIndex = ii;
      this.draft = this.todos[ii].desc;
      setTimeout(() => this.$refs.draft[0].focus(), 0);
    }
  },
  computed: {
    multiply() {
      return this.count * this.count2;
    },
    finished() {
      return this.todos.filter(t => t.isDone);
    },
    unfinished() {
      return this.todos.filter(function(t) {
        return t.isDone === false;
      });
    }
  }
};
</script>
