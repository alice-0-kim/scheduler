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
    <table>
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

    <hr>

    <!-- A collection of items per day -->
    <div class="today">
      <div v-for="(item, idx) in todos" :key="idx">
        <p>{{ item.todo }}</p>
      </div>
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
</style>
