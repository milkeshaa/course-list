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
      <div class="text" v-if="prototypeShow">
        Прообраз заданного рёберного графа:
      </div>
      <ul>
        <li v-for="vertex in list">
          {{ vertex.key }} -> {{ vertex.adjacent }}
        </li>
      </ul>
      <button class="button" @click="alghoritm" v-if="!prototypeShow">Обработать алгоритмом</button>
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
        prototypeShow: false,
        firstVertex: '',
        secondVertex: '',
        list: [{
          key: '',
          adjacent: []
        }],
        listOnSet: [{
          key: '',
          adjacent: []
        }],
        graph: {
          edges: [],
          verticies: []
        },
        prototype: {
          edges: [],
          verticies: []
        },
        graphCopy: null,
        triangle: null,
        oddTriangle: null,
        clique: {
          verticies: []
        },
        cover: []
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
        this.list.push({ key: firstVertex, adjacent: [secondVertex] });
      },
      addToListOnSet (firstVertex, secondVertex) {
        let object = this.listOnSet.find(element => {
          return element.key === firstVertex || !element.key;
        });
        if (object) {
          object.key = firstVertex;
          object.adjacent.push(secondVertex);
          return;
        }
        this.listOnSet.push({ key: firstVertex, adjacent: [secondVertex] });
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
        let triangle = {
          verticies: []
        }
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
              triangle.verticies = [u, v, vertex];
              flag = true;
            }
          });
        });
        return flag ? triangle : false;
      },
      expandClique (enviroment) {
        if (!enviroment.length) {
          return;
        }
        let counter = 0;
        let leftVerticies = enviroment.filter(vertex => {
          return !this.clique.verticies.includes(vertex);
        });
        leftVerticies.forEach(vertex => {
          this.clique.verticies.forEach(cliqueVertex => {
            if (this.graph.edges.includes(vertex + "_" + cliqueVertex)
                    || this.graph.edges.includes(cliqueVertex + "_" + vertex)) {
              counter++;
            }
          })
          if (counter === this.clique.verticies.length) {
            this.clique.verticies.push(vertex);
            this.expandClique(leftVerticies);
          }
        });
      },
      updateGraph () {
        this.graph.edges = this.graph.edges.filter(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          if (!(this.clique.verticies.includes(u) && this.clique.verticies.includes(v))) {
            return true;
          }
        });
        this.graph.verticies = this.graph.verticies.filter(vertex => {
          for (let i = 0; i < this.graph.edges.length; i++) {
            let edge = this.graph.edges[i];
            let delimiterIndex = edge.indexOf("_");
            let u = edge.substring(0, delimiterIndex);
            let v = edge.substring(delimiterIndex + 1);
            if (vertex === u || vertex === v) {
              return true;
            }
          }
        });
      },
      updateList () {
        this.list = [{
          key: '',
          adjacent: []
        }];
        this.graph.edges.forEach(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          this.addToList(u, v);
          this.addToList(v, u);
        });
      },
      expandCover () {
        let vertexInGraphAndFragment = this.graph.verticies.find(vertex => {
          for (let i = 0; i < this.cover.length; i++) {
            let fragment = this.cover[i];
            if (fragment.verticies.includes(vertex)) {
              return true;
            }
          }
        });
        if (vertexInGraphAndFragment) {
          this.findClique(vertexInGraphAndFragment);
          this.cover.push(this.clone(this.clique));
        }
      },
      findClique (vertex) {
        let currentVertex = this.list.find(element => {
          return element.key === vertex;
        });
        if (currentVertex.adjacent.length === 1) {
          this.clique.verticies = currentVertex.adjacent;
        } else if (currentVertex.adjacent.length === 2) {
          if (this.graph.edges.includes((currentVertex.adjacent[0] + "_" + currentVertex.adjacent[1]))
                  || this.graph.edges.includes(currentVertex.adjacent[1] + "_" + currentVertex.adjacent[0])) {
            this.clique.verticies = currentVertex.adjacent;
          } else {
            alert("Граф не рёберный");
            return false;
          }
        } else if (currentVertex.adjacent.length > 2) {
          this.clique = this.findTriangle(currentVertex.adjacent);
          if (!this.clique) {
            alert("Граф не рёберный");
            return false;
          }
          this.expandClique(currentVertex.adjacent);
        }
        this.clique.verticies.push(currentVertex.key);
      },
      clone (object) {
          let copy = {};

          if (null === object || "object" != typeof object) return object;

          if (object instanceof Array) {
              copy = [];
              for (let i = 0, len = object.length; i < len; i++) {
                  copy[i] = this.clone(object[i]);
              }
              return copy;
          }

          if (object instanceof Object) {
              copy = {};
              for (var attr in object) {
                  if (object.hasOwnProperty(attr)) copy[attr] = this.clone(object[attr]);
              }
              return copy;
          }
      },
      updateCover () {
        let verticies = this.graphCopy.verticies.filter(vertex => {
          let counter = 0;
          for (let i = 0; i < this.cover.length; i++) {
            let fragment = this.cover[i];
            if (fragment.verticies.includes(vertex)) {
              counter++;
            }
          }
          if (counter < 2) {
            return true;
          }
        });
        verticies.forEach(vertex => {
          this.cover.push({ verticies: [vertex] });
        });
        this.createPrototype();
      },
      createPrototype () {
        for (let i = 0; i < this.cover.length; i++) {
          this.prototype.verticies.push(i + 1);
        }
        this.cover.forEach(fragment => {
          let intersection = this.cover.filter(element => {
            for (let i = 0; i < element.verticies.length; i++) {
              if (JSON.stringify(fragment.verticies) !== JSON.stringify(element.verticies)) {
                if (fragment.verticies.includes(element.verticies[i])) {
                  return true;
                }
              }
            }
          });
          if (intersection.length) {
            intersection.forEach(element => {
              if (!this.prototype.edges.includes((this.cover.indexOf(element) + 1) + "_" + (this.cover.indexOf(fragment) + 1))) {
                this.prototype.edges.push((this.cover.indexOf(fragment) + 1) + "_" + (this.cover.indexOf(element) + 1));
              }
            });
          }
        });
        this.prototype.edges.forEach(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          this.addToList(u, v);
          this.addToList(v, u);
        });
      },
      findIsolatedVertex (set) {
        let isolatedVertex = this.list.find(element => {
          for (let i = 0; i < set.length; i++) {
            if (element.adjacent.length === 1 && set[i] === element.key) {
              return true;
            }
          }
        });
        return isolatedVertex;
      },
      workWithTriangle (vertex) {
        this.clique.verticies = this.triangle.verticies;
        this.expandClique(vertex.adjacent);
        this.clique.verticies.push(vertex.key);
      },
      findOddTriangle (set) {
        let flag = false;
        let graphOnSet = {
          edges: []
        };
        let oddTriangle = null;
        this.graph.edges.forEach(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          if (set.includes(u) && set.includes(v)) {
            graphOnSet.edges.push(edge);
            this.addToListOnSet(u, v);
            this.addToListOnSet(v, u);
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
              let triangle = {
                verticies: [u, v, vertex]
              };
              let theRestOfSet = set.filter(Vertex => !triangle.verticies.includes(Vertex));
              let oddVertex = this.listOnSet.find(element => {
                for (let i = 0; i < theRestOfSet.length; i++) {
                  if (element.key === theRestOfSet[i]) {
                    if (element.adjacent.includes(u, v, vertex)) {
                      return true;
                    }
                    if ((element.adjacent.includes(u) && !element.adjacent.includes(v) && !element.adjacent.includes(vertex))
                            || (element.adjacent.includes(vertex) && !element.adjacent.includes(v) && !element.adjacent.includes(u))
                            || (element.adjacent.includes(v) && !element.adjacent.includes(u) && !element.adjacent.includes(vertex))) {
                      return true;
                    }
                  }
                }
              });
              if (oddVertex) {
                oddTriangle = triangle;
                flag = true;
                return;
              }
            }
          });
        });
        return flag ? oddTriangle : false;
      },
      isNotIsomorphicToCycle (set) {
        let graphOnSet = {
          edges: []
        };
        this.listOnSet = [{
          key: '',
          adjacent: []
        }];
        this.graph.edges.forEach(edge => {
          let delimiterIndex = edge.indexOf("_");
          let u = edge.substring(0, delimiterIndex);
          let v = edge.substring(delimiterIndex + 1);
          if (set.includes(u) && set.includes(v)) {
            graphOnSet.edges.push(edge);
            this.addToListOnSet(u, v);
            this.addToListOnSet(v, u);
          }
        });
        if (graphOnSet.edges.length !== 4) {
          return false;
        }
        let flag = this.listOnSet.find(element => { element.adjacent.length !== 2 });
        return flag;
      },
      workWithFive (vertex) {
        let subset = [];
        for (let i = 0; i < SUBSET_OF_5; i++) {
          subset.push(vertex.adjacent[i]);
        }
        this.triangle = this.findTriangle(subset);
        if (!this.triangle) {
          alert("Граф не рёберный");
          return;
        }
        this.workWithTriangle(vertex);
      },
      workWithFour (vertex) {
        let isolatedVertex = this.findIsolatedVertex(vertex.adjacent);
        if (!isolatedVertex) {
          this.triangle = this.findTriangle(vertex.adjacent);
          if (!this.triangle) {
            let set = this.clone(vertex.adjacent);
            set.push(vertex.key);
            this.oddTriangle = this.findOddTriangle(set);
            if (this.oddTriangle) {
              console.log(this.oddTriangle, 'odd triangle');
              this.clique = this.clone(this.oddTriangle);
              return;
            }
            if (this.isNotIsomorphicToCycle(vertex.adjacent)) {
              alert("Граф не рёберный");
              return;
            }
            let supList = [];
            this.list.forEach(element => {
              if (vertex.adjacent.includes(element.key)) {
                supList.push(element);
              }
            });
            let Vertex = supList.find(element => element.adjacent.length > 3);
            console.log(Vertex);
            if (!Vertex) {
              let takenVerticies = [];
              for (let i = 0; i < vertex.adjacent.length - 1; i++) {
                if (this.graph.edges.includes((vertex.adjacent[i] + "_" + vertex.adjacent[i + 1])
                        || (vertex.adjacent[i + 1] + "_" + vertex.adjacent[i]))) {
                  takenVerticies.push(vertex.adjacent[i + 1]);
                  takenVerticies.push(vertex.adjacent[i]);
                  break;
                }
              }
              this.clique.verticies = takenVerticies;
              this.clique.verticies.push(vertex.key);
              this.cover.push(this.clone(this.clique));
              this.updateGraph();
              this.updateList();
              this.clique.verticies = [];
              let lastVerticies = vertex.adjacent.filter(v => !takenVerticies.includes(v));
              this.clique.verticies = lastVerticies;
              this.clique.verticies.push(vertex.key);
              return;
            }
            let aVertex = this.list.find(element => {
              return Vertex.adjacent.includes(element.key) && !vertex.adjacent.includes(element.key) && element.key !== vertex.key;
            });
            let aVertexAdjacnet = [];
            vertex.adjacent.forEach(v => {
              if (aVertex.adjacent.includes(v)) {
                aVertexAdjacnet.push(v);
              }
            });
            if (aVertexAdjacnet.length === 1 || aVertexAdjacnet.length === 3) {
              alert("Граф не рёберный");
              return;
            } else if (aVertexAdjacnet.length === 2) {
              if (this.graph.edges.includes(aVertexAdjacnet[0] + "_" + aVertexAdjacnet[1])
                      || this.graph.edges.includes(aVertexAdjacnet[1] + "_" + aVertexAdjacnet[0])) {
                this.clique.verticies.push(aVertexAdjacnet[1], aVertexAdjacnet[0], aVertex.key);
                return;
              }
              this.triangle = this.findTriangle(set);
              if (!this.triangle) {
                alert("Граф не рёберный");
                return;
              }
              this.clique = this.clone(this.triangle);
            } else if (aVertexAdjacnet.length === 4) {
              this.triangle = this.findTriangle(set);
              if (!this.triangle) {
                alert("Граф не рёберный");
                return;
              }
              this.clique = this.clone(this.triangle);
            }
          }
          this.workWithTriangle(vertex);
        } else {
          this.clique.verticies.push(vertex.key);
          this.clique.verticies.push(isolatedVertex.key);
        }
      },
      workWithThree (vertex) {
        console.log(this.findTriangle(vertex.adjacent));
      },
      alghoritm () {
        this.prototypeShow = true;
        this.graphCopy = this.clone(this.graph);
        let vertexOfMaxDegree = this.getVertexOfMaxDegree();
        if (vertexOfMaxDegree.adjacent.length >= 5) {
          this.workWithFive(vertexOfMaxDegree);
        }
        if (vertexOfMaxDegree.adjacent.length === 4) {
          this.workWithFour(vertexOfMaxDegree);
        }
        if (vertexOfMaxDegree.adjacent.length === 3) {
          this.workWithThree(vertexOfMaxDegree);
        }
        this.cover.push(this.clone(this.clique));
        while (this.graph.verticies.length && this.graph.edges.length) {
          this.updateGraph();
          this.updateList();
          this.expandCover();
        }
        console.log(this.cover);
        this.updateCover();
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
    cursor: pointer;
  }
  html, body {
    margin: 0;
    height: 100%;
    width: 100%;
  }
</style>
