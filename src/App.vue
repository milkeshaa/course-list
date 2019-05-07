<template>
  <div id="app" class="container">
    <div class="form" v-if="!show">
      <div class="edge">
        Введите ребро, если же вершина одна, второе поле оставьте пустым.
        <input placeholder="Введите вершину" v-model="firstVertex" type="text"/>
        <input placeholder="Введите вершину" v-model="secondVertex" type="text"/>
        <button @click="addEdge">Добавить ребро в граф</button>
      </div>
    </div>
    <div v-else class="graph">
      <ul>
        <li v-for="vertex in graph.list">
          {{ vertex.key }} -> {{ vertex.values }}
        </li>
      </ul>
    </div>
    <button @click="show = !show" class="button button_switcher">Показать\Спрятать граф</button>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        show: false,
        firstVertex: '',
        secondVertex: '',
        graph: {
          edges: [],
          verticies: [],
          list: [{
            key: '',
            values: []
          }]
        },
      }
    },
    methods: {
      hasVertex (vertex) {
        return this.graph.verticies.includes(vertex);
      },
      addVertex (vertex) {
        this.graph.verticies.push(vertex);
      },
      addToList (firstVertex, secondVertex) {
        let object = this.graph.list.find(element => {
          return element.key === firstVertex || !element.key;
        });
        if (object) {
          object.key = firstVertex;
          object.values.push(secondVertex);
          return;
        }
        this.graph.list.push({ key: firstVertex, values: [...secondVertex] });
      },
      addEdge () {
        if (!this.hasVertex(this.firstVertex)) {
          this.addVertex(this.firstVertex);
        }
        if (!this.hasVertex(this.secondVertex)) {
          this.addVertex(this.secondVertex);
        }
        this.addToList(this.firstVertex, this.secondVertex);
        this.addToList(this.secondVertex, this.firstVertex);
        this.graph.edges.push(this.firstVertex + "_" + this.secondVertex);
        this.renewVerticies();
      },
      renewVerticies () {
        this.firstVertex = '';
        this.secondVertex = '';
      }
    }
  }
</script>

<style lang="scss">
  .container {
    background-color: plum;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    height: 100%;
    width: 100%;
  }
  .game {
    text-align: center;
    font-size: 72px;
    border: aqua 1px solid;
    border-radius: 12px;
  }
  .button {
    margin: 5px;
    padding: 10px;
    font-size: 16px;
  }
  html, body {
    margin: 0;
    height: 100%;
    width: 100%;
  }
</style>
