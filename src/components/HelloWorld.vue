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

    <!-- Calendar -->
    <div class="container calendar">
      <h2>Calendar</h2>
      <h3>{{ year }} | {{ month }}</h3>
      <div class="cell day" v-for="a in days" :key="a"><p>{{ a }}</p></div>
      <div class="cell" v-for="b in pre" :key="b + '-pre'"></div>
      <div class="cell" v-for="c in 31" :key="c" :id="c"><p>{{ c }}</p></div>
      <div class="cell" v-for="d in post" :key="d + '-post'"></div>
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
      todo: '',
      year: 2018,
      month: 6,
      days: ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'],
      pre: 0,
      post: 4
    }
  },
  watch: {
    items: function (arr) {
      this.updateCalendar(arr)
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
    },
    updateCalendar (arr) {
      for (var i of arr) {
        var d = new Date(parseInt(i.createdAt.seconds) * 1000)
        var y = d.getFullYear()
        var m = d.getMonth()
        var t = d.getDate()
        if (this.isOnCalendar(y, m)) {
          var elem = document.createElement('p')
          var text = document.createTextNode(i.todo)
          elem.appendChild(text)
          elem.className = i.done ? 'success todo' : 'caution todo'
          document.getElementById(t).appendChild(elem)
        }
      }
    },
    isOnCalendar (y, m) {
      return y === this.year && m === this.month
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
/* ----- calendar ----- */
.calendar {
  display: block;
  clear: both;
  width: calc(100% - 40px);
  //font-size: 0;
}
.calendar h2 {
  font-size: 24px;
  margin: inherit;
}
.calendar .cell {
  display: inline-block;
  font-size: 1em;
  width: calc((100% - 15px) / 7);
  height:200px;
  /* background:yellowgreen; */
  border: 1px solid gray;
  font-size: 15px;
  text-align: left;
  float: left;
}
.calendar .cell.day {
  height: auto;
  text-align: center;
}
.calendar .cell p {
  padding: 0 10px;
}
</style>
<style>
.todo {
  padding: 0 10px;
  margin: 5px 0;
}
.caution {
  background: #ff6961;
  /*box-shadow: 1px 1px 1px 1px rgba(255,105,97,0.2);*/
}
.success {
  background: #2aab2a;
  /*box-shadow: 1px 1px 1px 1px rgba(58,206,58,0.2);*/
}
</style>
