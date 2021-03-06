<template>
  <section class="container note-wrap">
    <div class="card mb-3 note">
      <div class="card-title note-title text-left d-flex align-center justify-between bg-secondary">
        <h3 class="m-0">{{note.name | replaceText(noteNameMaxSymbol)}}</h3>
        <div class="note-title-action">
          <Icon
            v-for="icon in icons"
            :key="icon.symbol"
            :symbol="icon.symbol"
            :class-name="icon.className"
            @action="modalOpen(icon.modalContentIndex)"
            class="mr-2"
          ></Icon>
        </div>
      </div>
      <div class="card-body note-body">
        <InputAction
          :placeholder="inputOption.placeholder"
          :text="inputOption.text"
          :value="inputOption.value"
          :disabled="!todoList.mutable"
          @submit="inputSubmit"
          @input="val => inputOption.value = val"
        >
        </InputAction>
        <ToDoList
          :id="index.toString()"
          :list="note.todoList"
          :mutable="todoList.mutable"
          @action="toDoListAction"
          @action:edit="toDoListActionEdit"
          @action:remove="toDoListActionRemove"
        ></ToDoList>
        <p v-if="note.todoList.length === 0" class="m-0">Empty todo list</p>
      </div>
    </div>
    <Modal
      v-if="modalActive"
      :active="modalActive"
      @close="modalClose"
    >
      <template slot="header">
        <h2 class="m-0">{{modalCurrentContent.header}}:</h2>
      </template>
      <template slot="body">
        <h2>{{modalCurrentContent.body}}</h2>
      </template>
      <template slot="footer">
        <button
          v-for="(button, i) in modalCurrentContent.footer.buttons"
          :key="i"
          :class="{[button.className]: button.className}"
          @click="button.handler(index, note)"
          type="button"
          class="m-2 mt-0 mb-0"
        >
          {{button.text}}
        </button>
      </template>
    </Modal>
  </section>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex'
import InputAction from '@/components/ui/Input'
export default {
  name: 'Note',
  components: {
    InputAction
  },
  data () {
    return {
      todoList: {
        mutable: true
      },
      noteNameMaxSymbol: 36,
      noteTemp: null,
      icons: [
        {
          symbol: 'fa fa-share',
          className: 'text-primary',
          modalContentIndex: 3
        },
        {
          symbol: 'fa fa-pencil',
          className: 'text-primary',
          modalContentIndex: 2
        },
        {
          symbol: 'fa fa-floppy-o',
          className: 'text-success',
          modalContentIndex: 0
        },
        {
          symbol: 'fa fa-times',
          className: 'text-danger',
          modalContentIndex: 1
        }
      ],
      modalActive: false,
      modalCurrentContent: null,
      modalContent: [
        {
          header: 'Success',
          body: 'Your note will be saved!',
          footer: {
            buttons: [
              {
                text: 'Accept',
                handler: this.saveNote,
                className: 'btn-success'
              }
            ]
          }
        },
        {
          header: 'Question',
          body: 'Are you sure you want to delete the note?',
          footer: {
            buttons: [
              {
                text: 'Cancel',
                handler: this.modalClose,
                className: 'btn-primary'
              },
              {
                text: 'Accept',
                handler: this.removeFromNotesList,
                className: 'btn-success'
              }
            ]
          }
        },
        {
          header: 'Question',
          body: 'You will not be able to edit Todo list?',
          footer: {
            buttons: [
              {
                text: 'Cancel',
                handler: this.modalClose,
                className: 'btn-primary'
              },
              {
                text: 'Accept',
                handler: this.changeEditTodoList,
                className: 'btn-success'
              }
            ]
          }
        },
        {
          header: 'Question',
          body: 'Undo the change?',
          footer: {
            buttons: [
              {
                text: 'Cancel',
                handler: this.modalClose,
                className: 'btn-primary'
              },
              {
                text: 'Accept',
                handler: this.undoChangeNote,
                className: 'btn-success'
              }
            ]
          }
        }
      ],
      inputOptionAdd: {
        placeholder: 'Please enter todo list name',
        text: 'Add',
        value: ''
      },
      inputOptionEdit: {
        placeholder: 'Please change todo list name',
        text: 'Save',
        value: ''
      },
      inputActionEdit: {
        apply: false,
        index: null
      }
    }
  },
  computed: {
    ...mapGetters(['noteCurrent']),
    index () {
      const radix = 10
      return parseInt(this.$route.params.id, radix) - 1
    },
    note () {
      return this.noteCurrent
    },
    inputOption () {
      if (this.inputActionEdit.apply) {
        return this.inputOptionEdit
      }
      return this.inputOptionAdd
    }
  },
  methods: {
    ...mapMutations(['updateNote', 'removeNote', 'writeNoteCurrent']),
    toDoListAction (list = []) {
      console.log(list)
    },
    toDoListActionEdit ({ list, index }) {
      this.inputOptionEdit.value = list[index].title
      this.inputActionEdit.index = index
      this.inputActionEdit.apply = true
    },
    toDoListActionRemove ({ list, index }) {
      this.inputOptionAdd.value = this.inputOptionEdit.value = ''
      this.inputActionEdit.apply = false
      list.splice(index, 1)
    },
    saveNoteTemp (note) {
      this.noteTemp = JSON.parse(JSON.stringify(note))
    },
    saveNote () {
      const noteData = { index: this.index, note: this.note }
      this.updateNote(noteData)
      this.modalClose()
    },
    removeFromNotesList (index) {
      this.removeNote({ index })
      this.modalClose()
      this.$router.push({ name: 'Main' })
    },
    undoChangeNote () {
      const note = this.noteTemp
      this.writeNoteCurrent(note)
      this.modalClose()
    },
    changeEditTodoList () {
      this.toggleEditTodoList()
      this.modalClose()
    },
    toggleEditTodoList () {
      this.todoList.mutable = !this.todoList.mutable
    },
    modalOpen (index) {
      this.modalCurrentContent = this.modalContent[index]
      if (this.modalCurrentContent) {
        this.modalActive = true
      }
    },
    modalClose () {
      this.modalActive = false
    },
    inputSubmit (value) {
      if (!this.todoList.mutable) return
      // edit item in todolist
      if (this.inputActionEdit.apply) {
        const index = this.inputActionEdit.index
        this.note.todoList[index].title = value
        this.inputActionEdit.apply = false
        return
      }
      // added item in todolist
      const newTodoItem = {
        status: false,
        title: value
      }
      this.note.todoList.unshift(newTodoItem)
      this.inputOptionAdd.value = ''
    }
  },
  created () {
    this.saveNoteTemp(this.note)
  }
}
</script>

<style scoped lang="scss">
  .note-wrap {
    .note {
      width: 100%;
    }
  }
</style>
