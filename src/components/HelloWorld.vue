<template>
  <div class="hello">

    <!-- Add a new item -->
    <form @submit="addItem(todo)">
      <h2>Add a New Item</h2>
      <input v-model="todo" placeholder="What needs to be done?" class="input">
      <button type="submit" class="button success">Add New Item</button>
    </form>

    <hr>

    <!-- A collection of items -->
    <div class="container">
      <table class="table">
        <tr>
          <th>Date</th>
          <th>Description</th>
          <th>Done?</th>
          <th>Delete?</th>
        </tr>
        <tr v-for="(item, idx) in items" :key="idx">
          <td>
            <p>{{ item.createdAt | format }}</p>
          </td>
          <td>
            <p v-bind:class="{ progress: !item.done }">{{ item.todo }}</p>
          </td>
          <td>
            <input type="checkbox" :checked="item.done" @click="updateStatus(item.id, item.done)">
          </td>
          <td>
            <button class="button caution" @click="deleteItem(item.id)">
              Delete
            </button>
          </td>
        </tr>
      </table>
    </div>

    <!-- A collection of items per day -->
    <div class="container">
      <h2>To-do List</h2>
      <div v-for="(item, idx) in todos" :key="idx">
        <p>{{ item.todo }}</p>
      </div>
      <p v-if="todos.length === 0">You're awesome <font-awesome-icon icon="coffee" /></p>
    </div>
  </div>
</template>

<script>

import { db } from '../main'

export default {
  name: 'HelloWorld',
  data () {
    return {
      items: [],
      todos: [],
      todo: ''
    }
  },
  firestore () {
    return {
      items: db.collection('items').orderBy('createdAt'),
      todos: db.collection('items').where('done', '==', false).orderBy('createdAt')
    }
  },
  methods: {
    addItem (todo) {
      const createdAt = new Date()
      const done = false
      db.collection('items').add({ createdAt, todo, done })
      this.todo = ''
    },
    deleteItem (id) {
      db.collection('items').doc(id).delete()
    },
    updateStatus (id, status) {
      db.collection('items').doc(id).update({ done: !status })
    }
  },
  filters: {
    format (value) {
      if (value.length === 0) return ''
      var d = new Date(parseInt(value.seconds) * 1000)
      return d.toLocaleDateString() + ' ' + d.toLocaleTimeString()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.progress {
  color: red;
}
* {
  outline: none;
}
*:focus {
  outline: none;
}
/* ----- table ----- */
.table {
  width: 100%;
}
/* ----- hello ----- */
.hello .input {
  border: none;
  box-shadow: 1px 1px 1px 1px rgba(0,0,0,0.2);
  transition: all ease 0.3s;
  width: 500px;
  height: 30px;
  margin: 0 20px 20px 0;
}
.hello .input:hover, .hello .button:hover {
  transform: translateY(-2px);
  box-shadow: 1px 2px 3px 3px rgba(0,0,0,0.2);
}
.hello .button {
  border: none;
  border-radius: 5px;
  box-shadow: 1px 1px 1px 1px rgba(0,0,0,0.2);
  transition: all ease 0.3s;
}
.hello .button.caution {
  background: #ff6961;
  /*box-shadow: 1px 1px 1px 1px rgba(255,105,97,0.2);*/
}
.hello .button.success {
  background: #2aab2a;
  /*box-shadow: 1px 1px 1px 1px rgba(58,206,58,0.2);*/
}
/* ----- container ----- */
.container {
  display: inline-block;
  float: left;
  width: calc(50% - 40px);
  padding: 20px;
  margin: 20px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  box-shadow: 1px 1px 1px 1px rgba(0,0,0,0.2);
}
.container h2 {
  margin: 0;
}
</style>
