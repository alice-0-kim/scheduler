<template>
  <div class="calendar">
  </div>
</template>

<script>

import { db } from '../main'

export default {
  name: 'Calendar',
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
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.calendar {
  display: block;
  clear: both;
  width: calc(100% - 40px);
  font-size: 0;
  display: inline-block;
  float: left;
  width: calc(50% - 40px);
  padding: 20px;
  margin: 20px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  box-shadow: 1px 1px 1px 1px rgba(0,0,0,0.2);
}
.calendar h2 {
  font-size: 24px;
  margin: 0;
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
.calendar .cell p{
  padding: 0 10px;
}
</style>
