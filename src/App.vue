<template>
  <div id="app" class="container">
    <div class="form" v-if="!show">
      <div class="edge">
        <div class="text">
          Введите ребро, если же вершина одна, второе поле оставьте пустым.
        </div>
        <div class="inputs-container">
          <input class="input" placeholder="Введите вершину" v-model="firstVertex" type="text"/>
          <input class="input" placeholder="Введите вершину" v-model="secondVertex" type="text"/>
          <button class="button" @click="addEdge">Добавить ребро в граф</button>
        </div>
      </div>
    </div>
    <div v-else class="graph">
      <ul>
        <li v-for="vertex in list">
          {{ vertex.key }} -> {{ vertex.adjacent }}
        </li>
      </ul>
      <button class="button" @click="alghoritm">Обработать алгоритмом</button>
    </div>
    <button @click="show = !show" class="button button_switcher">Показать\Спрятать граф</button>
  </div>
</template>

<script>
  const SUBSET_OF_5 = 5;

  export default {
    data () {
      return {
        show: false,
        firstVertex: '',
        secondVertex: '',
        list: [{
          key: '',
          adjacent: []
        }],
        graph: {
          edges: [],
          verticies: []
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
        let object = this.list.find(element => {
          return element.key === firstVertex || !element.key;
        });
        if (object) {
          object.key = firstVertex;
          object.adjacent.push(secondVertex);
          return;
        }
        this.list.push({ key: firstVertex, adjacent: [...secondVertex] });
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
      },
      getVertexOfMaxDegree () {
        let max = this.list[0].adjacent.length;
        let index = 0;
        for (let i = 0; i < this.list.length; i++) {
          if (this.list[i].adjacent.length > max) {
            max = this.list[i].adjacent.length;
            index = i;
          }
        }
        return this.list[index];
      },
      findTriangle (set) {
        let flag = false;
        let graphOnSet = {
          edges: []
        };
        this.graph.edges.forEach(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          if (set.includes(u) && set.includes(v)) {
            graphOnSet.edges.push(edge);
          }
        });
        if (graphOnSet.edges.length < 3) {
          return false;
        }
        graphOnSet.edges.forEach(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          set.forEach(vertex => {
            if ((graphOnSet.edges.includes(v + "_" + vertex) && graphOnSet.edges.includes(vertex + "_" + u))
                || (graphOnSet.edges.includes(vertex + "_" + v) && graphOnSet.edges.includes(u + "_" + vertex))) {
              flag = true;
            }
          });
        });
        return flag;
      },
      workWithFive (vertex) {
        let subset = [];
        for (let i = 0; i < SUBSET_OF_5; i++) {
          subset.push(vertex.adjacent[i]);
        }
        console.log(this.findTriangle(subset));
      },
      alghoritm () {
        let vertexOfMaxDegree = this.getVertexOfMaxDegree();
        if (vertexOfMaxDegree.adjacent.length >= 5) {
          this.workWithFive(vertexOfMaxDegree);
        }
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
  .inputs-container {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
  }
  .edge {
    text-align: center;
    flex-direction: column;
    font-size: 24px;
  }
  .input {
    border-radius: 5px;
    margin: 3px;
    text-align: center;
    width: 50%;
  }
  .button {
    margin: 5px;
    padding: 10px;
    font-size: 16px;
    border-radius: 10px;
  }
  html, body {
    margin: 0;
    height: 100%;
    width: 100%;
  }
</style>
