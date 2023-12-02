<template>
  <div>
    <span class="button" @click="getData()">Get Data</span>
    <span class="button" @click="addRandomElement()">Add Random Element</span>
    <select @change="onInstanceSelected" v-model="selected">
      <option v-for="(item,key) in scenarios" :key="key" :value="item.value">{{ item.label }}</option>
    </select>
    <div id="cy"></div>
    <pre>{{ selected }}</pre>
  </div>
</template>
<style type="text/css">
#cy{
  width:90vw;
  height:90vh;
  display:block;
  border:1px solid #cecece;
}
.button{
  display:inline-block;
  border:1px solid #cecece;
  width:150px;
  padding:10px;
  text-align: center;
  margin:10px;
  border-radius: 10px;
  cursor: pointer;
}
select{
  display:inline-block;
  border:1px solid #cecece;
  min-width:150px;
  padding:10px;
  text-align: center;
  margin-bottom:10px;
  border-radius: 10px;
  cursor: pointer;
}
</style>
<script>

import cytoscape from 'cytoscape/dist/cytoscape.esm';
import dagre from 'cytoscape-dagre';
import elk from 'cytoscape-elk';
import klay from 'cytoscape-klay';
import 'cytoscape-panzoom/cytoscape.js-panzoom.css'
import panzoom from 'cytoscape-panzoom/cytoscape-panzoom.js';

export default{
  components:{

  },
  mounted() {
    cytoscape.use(dagre);
    cytoscape.use( elk );
    cytoscape.use( klay );
    cytoscape.use(panzoom)

    
    let data = this.scenarios.find((item)=> item.value == 4).data
    this.initCytoscape(data)
  },
  methods: {
    getData(){
      console.log("data",this.cy.data())
    },
    initCytoscape(_elements){
      console.log("_elements",_elements)
      if(this.cy != null){
        let current_elements = this.cy.elements();
        this.cy.remove(current_elements)
        console.log("hello")
      }
      this.cy = new cytoscape({
        container: document.getElementById("cy"),
        
        elements:_elements,  
        boxSelectionEnabled: true,
        style: [ // the stylesheet for the graph
          {
            selector: '.node',
            style: {
              'label': 'data(id)',
              "text-valign":"bottom",
              "font-size": 3,
              'background-image':['https://cdn4.iconfinder.com/data/icons/internet-security-flat-2/32/Internet_Security_Router_signal_wifi_lock_password-256.png'],
              "background-width":'80%',
              "background-height":'80%',
              "background-color": "#FFF"
            }
          },
          {
            selector: '.upstream',
            style: {
              'width': 1,
              "font-size": 5,
              'label': 'data(id)',
              'line-color': '#ccc',
              'source-arrow-color': '#ccc',
              'source-arrow-shape': 'triangle',
              'curve-style': 'bezier',
              "source-text-offset":20,
              "target-text-offset":20
            }
          },
          {
            selector: '.downstream',
            style: {
              'width': 1,
              'line-color': '#ccc',
              'target-arrow-color': '#ccc',
              'target-arrow-shape': 'triangle',
              'curve-style': 'taxi'
            }
          },
          {
            selector: 'node',
            style: {
              'width': 10,
              'height': 10,
              'overlay-color':"#cecece",
              "overlay-shape": "ellipse"
            }
          },
          {
            selector: 'edge',
            style: {
              'width': 0.75,
              'font-size':3,
              'arrow-scale':0.25,
              'control-point-step-size':14,
              'text-wrap': 'wrap',
              'text-max-width':10,
              'overlay-padding':5,
            }
          }
        ],

        layout: {
          name: 'dagre',
          fit: false,
          rankDir: 'TB',
          minLen: 1,
          nodeDimensionsIncludeLabels: true,
          //elk: {
          //  algorithm: 'mrtree',
          //  edgeEndTextureLength: 10000
          //},
        },
        zoom: 5,
      });
      this.cy.on("select","node",(e)=>{
        let node = e.target;
        console.log("select",node.id())
        console.log("select",node.position())


      });
          var defaults = {
              zoomFactor: 0.05, // zoom factor per zoom tick
              zoomDelay: 45, // how many ms between zoom ticks
              minZoom: 0.1, // min zoom level
              maxZoom: 10, // max zoom level
              fitPadding: 50, // padding when fitting
              panSpeed: 10, // how many ms in between pan ticks
              panDistance: 10, // max pan distance per tick
              panDragAreaSize: 75, // the length of the pan drag box in which the vector for panning is calculated (bigger = finer control of pan speed and direction)
              panMinPercentSpeed: 0.25, // the slowest speed we can pan by (as a percent of panSpeed)
              panInactiveArea: 8, // radius of inactive area in pan drag box
              panIndicatorMinOpacity: 0.5, // min opacity of pan indicator (the draggable nib); scales from this to 1.0
              zoomOnly: true, // a minimal version of the ui only with zooming (useful on systems with bad mousewheel resolution)
              fitSelector: undefined, // selector of elements to fit
              animateOnFit: function(){ // whether to animate on fit
                return false;
              },
              fitAnimationDuration: 1000, // duration of animation on fit

              // icon class names
              sliderHandleIcon: 'fa fa-minus',
              zoomInIcon: 'fa fa-plus',
              zoomOutIcon: 'fa fa-minus',
              resetIcon: 'fa fa-expand'
            };

            // add the panzoom control
            this.cy.panzoom( defaults );

    },
    onInstanceSelected(){
      let data = this.scenarios.find((item)=> item.value == this.selected).data
      this.initCytoscape(data)
    },
    addRandomElement(){
      let _id =  Math.ceil(Math.random()*100).toString()
      let _edge_id =  Math.ceil(Math.random()*100).toString()
      this.cy.add({
        data:{ "id": _id}
      })
      this.cy.add({
        data:{ "id": _edge_id, "source": _id, "target":"b"}
      })
    },
  },
  data(){
    return {
      cy:null,
      message:"hello",
      selected:"",
      scenarios:[
        { 
          label:"Scenario 1", 
          value: 1,
          data: [
            { "group":"nodes", "data":{ "id": "UPSTREAM" } },
            { "group":"nodes", "classes":["node"],  "data":{ "id": "NODE" } },
            { "group":"nodes", "data":{ "id": "DOWNSTREAM" } },
            { 
              "group":"edges", 
              "classes":["upstream"],
              "data":{ 
                
                "id": "LOREM IPSUM LOREM IPSUM LOREM IPSUM LOREM IPSUM", 
                "source":"UPSTREAM", 
                "target": "NODE"
              }
          },
            { 
              "group":"edges", 
              "classes":["downstream"],
              "data":{ 
                "id": "NDS", 
                "source":"NODE",
                "target": "DOWNSTREAM" 
              }
            }
          ] 
        },
        { 
          label:"Scenario 2",
          value: 2,
          data: [
            { "group":"nodes", "data":{ "id": "UPSTREAM 1" } },
            { "group":"nodes", "data":{ "id": "UPSTREAM 2" } },
            { "group":"nodes", "classes":["node"], "data":{ "id": "NODE" } },
            { "group":"nodes", "data":{ "id": "DOWNSTREAM 1" } },
            { "group":"nodes", "data":{ "id": "DOWNSTREAM 2" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS2", "source":"UPSTREAM 2", "target": "NODE" } },
            { "group":"edges", classes:["downstream"], "data":{ "id": "NDS1", "source":"NODE", "target": "DOWNSTREAM 1" } },
            { "group":"edges", classes:["downstream"], "data":{ "id": "NDS2", "source":"NODE", "target": "DOWNSTREAM 2" } },
          ] 
        },
        { 
          label:"Scenario 3",          
          value: 3, 
          data: [
            { "group":"nodes", "data":{ "id": "UPSTREAM 1" } },
            { "group":"nodes", "data":{ "id": "UPSTREAM 2" } },
            { "group":"nodes", "classes":["node"], "data":{ "id": "NODE 1" } },
            { "group":"nodes", "classes":["node"], "data":{ "id": "NODE 2" } },
            { "group":"nodes", "data":{ "id": "DOWNSTREAM 1" } },
            { "group":"nodes", "data":{ "id": "DOWNSTREAM 2" } },
            { "group":"nodes", "data":{ "id": "DOWNSTREAM 3" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1", "source":"UPSTREAM 1", "target": "NODE 1" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS2", "source":"UPSTREAM 2", "target": "NODE 1" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS3", "source":"UPSTREAM 1", "target": "NODE 2" } },
            { "group":"edges", classes:["downstream"], "data":{ "id": "NDS1", "source":"NODE 1", "target": "DOWNSTREAM 1" } },
            { "group":"edges", classes:["downstream"], "data":{ "id": "NDS2", "source":"NODE 1", "target": "DOWNSTREAM 2" } },
            { "group":"edges", classes:["downstream"], "data":{ "id": "NDS3", "source":"NODE 2", "target": "DOWNSTREAM 3" } },
          ] 
        },
        { 
          label:"Scenario 4",
          value: 4,
          data: [
            { "group":"nodes", "classes":["node"], "data":{ "id": "UPSTREAM 1" } },
            { "group":"nodes", "classes":["node"], "data":{ "id": "UPSTREAM 2" } },
            { "group":"nodes", "classes":["node"], "data":{ "id": "NODE" } },
            { "group":"nodes", "classes":["node"], "data":{ "id": "DOWNSTREAM 1" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1.1", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1.2", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1.3", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1.4", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1.5", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS1.6", "source":"UPSTREAM 1", "target": "NODE" } },
            { "group":"edges", classes:["upstream"], "data":{ "id": "NUS2", "source":"UPSTREAM 2", "target": "NODE" } },
            { "group":"edges", classes:["downstream"], "data":{ "id": "NDS1", "source":"NODE", "target": "DOWNSTREAM 1" } },
          ] 
        },
      ]
    }
  }
}
</script>

<style scoped>
</style>
