<template>
  <main class="container mx-auto flex justify-between items-stretch h-screen">
    <textarea
      class="border p-2 flex-1"
      v-model="inputText"
      @keydown.enter="parseWhenEnter"
    />
    <div class="border p-2 flex-1" id="canvasDiv" />
  </main>
</template>

<script>
import { marked } from "marked";
import { Network } from "vis-network/peer/esm/vis-network";
import { DataSet } from "vis-data/peer/esm/vis-data";

export default {
  data() {
    return {
      inputText: "",
      parsedText: "",
      parsedHtml: {},
      ctx: null,
      width: 600,
      height: 400,
    };
  },
  methods: {
    uniq(array) {
      const uniquedArray = [];
      for (const elem of array) {
        if (
          uniquedArray.findIndex((val) => {
            console.log("label", elem.label, val.label)
            return elem.label == val.label;
          }) == -1
        )
          uniquedArray.push(elem);
      }
      return uniquedArray;
    },
    parseWhenEnter() {
      let myNodes = [];
      let myEdges = [];

      this.parsedText = marked.parse(this.inputText);
      console.log(this.parsedText);

      const parser = new DOMParser();
      this.parsedHtml = parser.parseFromString(this.parsedText, "text/html");
      console.log(this.parsedHtml);

      const parentItem = this.parsedHtml.querySelector("ul");
      console.log(parentItem);

      const firstLvListItems = parentItem.querySelectorAll(":scope > li");
      console.log(firstLvListItems);

      firstLvListItems.forEach((item) => {
        const sourceBf = item.textContent;
        const sre = RegExp("(.+)\n", "g");
        const smatches = [...sourceBf.matchAll(sre)];
        const source = smatches[0][1];
        myNodes.push({ id: myNodes.length, label: source });
        myNodes = this.uniq(myNodes);

        const firstLvListParent = item.querySelector(":scope > ul");
        const secondLvListItems =
          firstLvListParent.querySelectorAll(":scope > li");
        console.log(secondLvListItems);
        secondLvListItems.forEach((secondItem) => {
          const tcontent = secondItem.textContent;
          console.log(tcontent);
          const re = RegExp("(.+):(.+)", "g");
          const matches = [...tcontent.matchAll(re)];
          const target = matches[0][1].trim();
          const label = matches[0][2].trim();
          myNodes.push({ id: myNodes.length, label: target });
          myNodes = this.uniq(myNodes);

          myEdges.push({
            from: myNodes.findIndex((val) => {
              return val.label == source;
            }),
            to: myNodes.findIndex((val) => {
              return val.label == target;
            }),
            arrows: "to",
            width: 1,
            label: label,
          });
        });
      });

      var nodes = new DataSet(myNodes);
      var edges = new DataSet(myEdges);

      var container = document.getElementById("canvasDiv");
      var data = {
        nodes: nodes,
        edges: edges,
      };
      var options = {};
      var network = new Network(container, data, options);
    },
  },
  mounted() {
    this.ctx = document.getElementById("canvasDiv").getContext("2d");
  },
};
</script>

