<template>
  <div class="hello">

    <!-- Add a new item -->
    <form @submit="addItem(todo)">
      <h2>Add a New Item</h2>
      <input v-model="todo" placeholder="What needs to be done?" class="input">
      <button type="submit" class="button success animate">Add New Item</button>
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
            <button class="button caution animate" @click="deleteItem(item.id)">
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
      <div class="sub">
        <button class="button" @click="getPreviousMonth">&#10094;</button>
        <h3>{{ year }} | {{ month + 1 }}</h3>
        <button class="button" @click="getNextMonth">&#10095;</button>
      </div>
      <div class="cell day" v-for="a in days" :key="a"><p>{{ a }}</p></div>
      <div class="cell" v-for="b in pre" :key="b + '-pre'"></div>
      <div class="cell" v-for="c in end" :key="c" :id="year + '-' + month + '-' + c">
        <p>{{ c }}</p>
        <div></div>
      </div>
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
      end: '',
      pre: 0,
      post: 4
    }
  },
  created () {
    var d = new Date()
    this.year = d.getFullYear()
    this.month = d.getMonth()
    this.end = new Date(this.year, this.month + 1, 0).getDate()
    this.pre = new Date(this.year, this.month, 1).getDay()
    this.post = 6 - new Date(this.year, this.month, this.end).getDay()
    this.logDate()
  },
  watch: {
    items: function (arr) {
      this.updateCalendar(arr)
    }
  },
  updated: function () {
    this.$nextTick(function () {
      this.refreshCalendar(this.end)
      this.updateCalendar(this.items)
    })
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
    refreshCalendar (num) {
      for (var j = 1; j < num; j++) {
        var cell = document.getElementById(this.year + '-' + this.month + '-' + j)
        var temp = cell.childNodes[0]
        cell.innerHTML = ''
        cell.appendChild(temp)
      }
    },
    updateCalendar (arr) {
      for (var i of arr) {
        var d = new Date(parseInt(i.createdAt.seconds) * 1000)
        var y = d.getFullYear()
        var m = d.getMonth()
        var t = d.getDate()
        if (this.isOnCalendar(y, m)) {
          var cell = document.getElementById(this.year + '-' + this.month + '-' + t)
          var elem = document.createElement('p')
          var text = document.createTextNode(i.todo)
          elem.appendChild(text)
          elem.className = i.done ? 'done todo' : 'in-progress todo'
          cell.appendChild(elem)
        }
      }
    },
    getPreviousMonth () {
      if (this.month === 0) {
        this.year--
        this.month = 11
      } else {
        this.month--
      }
      this.end = new Date(this.year, this.month + 1, 0).getDate()
      this.pre = new Date(this.year, this.month, 1).getDay()
      this.post = 6 - new Date(this.year, this.month, this.end).getDay()
    },
    getNextMonth () {
      if (this.month === 11) {
        this.year++
        this.month = 0
      } else {
        this.month++
      }
      this.end = new Date(this.year, this.month + 1, 0).getDate()
      this.pre = new Date(this.year, this.month, 1).getDay()
      this.post = 6 - new Date(this.year, this.month, this.end).getDay()
    },
    logDate () {
      console.log(this.year)
      console.log(this.month)
      console.log(this.end)
      console.log(this.pre)
      console.log(this.post)
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
.animate {
  transition: all ease 0.3s;
}
.input {
  border: none;
  box-shadow: 1px 1px 1px 1px rgba(0,0,0,0.2);
  width: 500px;
  height: 30px;
  margin: 0 20px 20px 0;
}
.button {
  border: none;
  border-radius: 5px;
  box-shadow: 1px 1px 1px 1px rgba(0,0,0,0.2);
}
.animate:hover {
  transform: translateY(-2px);
  box-shadow: 1px 2px 3px 3px rgba(0,0,0,0.2);
}
.caution {
  background: #ff6961;
  /*box-shadow: 1px 1px 1px 1px rgba(255,105,97,0.2);*/
}
.success {
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
  color: #fff;
  padding: 0 10px;
  margin: 5px 0;
}
.todo.in-progress {
  background: #ff90d5;
}
.todo.done {
  background: #90d5ff;
}
.sub * {
  display: inline-block;
  width: fit-content;
  margin: 10px auto;
}
</style>
