<template>
  <div>
    <h1>{{ this.title }}</h1>
    <svg :class="`barchart-svg-${chartId}`" :viewBox="viewBox">

      <g :transform="`translate(${margin.left}, ${margin.top})`">
        <g class="plot"></g>
        <g class="x-axis" :transform="`translate(0, ${height})`"></g>
        <g class="y-axis"></g>
      </g>

      <g :transform="`translate(${margin.left + width / 2}, ${margin.top + margin.bottom + height})`">
        <text>{{ labelX }}</text>
      </g>
      <g :transform="`translate(${margin.left / 4}, ${margin.top + height / 2}) rotate(-90)`">
        <text>{{ labelY }}</text>
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "BarChart",
  props: {
    chartId: {
      required: true,
      type: String,
    },
    title: {
      type: String,
    },
    dataset: {
      required: true,
      type: Array,
    },
    attribX: {
      required: true,
      type: String,
    },
    labelX: {
        type: String
    },
    attribY: {
      required: true,
      type: String,
    },
    attribColor: {
      required: false,
      type: String,
    },
    labelY: {
        type: String
    },
    height: {
      required: true,
      type: Number,
    },
    width: {
      required: true,
      type: Number,
    },
  },
  data() {
    return {
      x: null,
      y: null,
      color: null,
      margin: {
          left: 75,
          top: 25,
          right: 20,
          bottom: 55,
      }
    };
  },
  watch: {
    dataset() {
      this.update();
    },
    attribColor() {
      this.update();
    }
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.x = d3.scaleBand();
      this.y = d3.scaleLinear();
      this.color = d3.scaleSequential();

      this.update();
    },
    update() {
      d3.select(`.barchart-svg-${this.chartId}`)
                .select('.plot')
                .selectAll('text')
                .remove()
      
      this.x
        .domain(d3.map(this.dataset, (d) => d[this.attribX]))
        .range([0, this.width])
        .padding(0.2);
      this.y
        .domain([0, d3.max(this.dataset, (d) => d[this.attribY])])
        .nice()
        .range([this.height, 0]);

      this.color.domain(d3.extent(this.dataset, d => d[this.attribColor]))
        .interpolator(d3.interpolate("#ccc4ff", "#4126ff"));

      this.renderBars();
      this.renderXAxis();
      this.renderYAxis();
      this.renderLegend();
    },
    renderBars() {
      d3.select(`.barchart-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("rect")
        .data(this.dataset)
        .join("rect")
        .transition()
        .duration(200)
        .attr("x", (d) => this.x(d[this.attribX]))
        .attr("y", (d) => this.y(d[this.attribY]))
        .attr("width", this.x.bandwidth())
        .attr("height", (d) => this.y(0) - this.y(d[this.attribY]))
        .style("fill", (d) => this.color(d[this.attribColor]))
        .style("fill-opacity", 0.9)

      d3.select(`.barchart-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("rect")
        .on('click', (event, d) => {
            this.itemSelected(d)
        });
    },
    renderXAxis() {
      let xAxis = d3.axisBottom(this.x);

      d3.select(`.barchart-svg-${this.chartId}`).select(".x-axis").call(xAxis).selectAll("text")
    .attr("transform", "translate(-10,0)rotate(-45)");
    },
    renderYAxis() {
      let yAxis = d3.axisLeft(this.y);

      d3.select(`.barchart-svg-${this.chartId}`).select(".y-axis").call(yAxis);
    },
    renderLegend() {
      d3.select(`.barchart-svg-${this.chartId}`)
        .append("rect")
        .attr('x', 385)
        .attr('y', 15)
        .attr('width', 25)
        .attr('height', 10)
        .attr('fill', "#ccc4ff")
      d3.select(`.barchart-svg-${this.chartId}`)
        .append("rect")
        .attr('x', 410)
        .attr('y', 15)
        .attr('width', 25)
        .attr('height', 10)
        .attr('fill', "#8a7aff")      
      d3.select(`.barchart-svg-${this.chartId}`)
        .append("rect")
        .attr('x', 435)
        .attr('y', 15)
        .attr('width', 25)
        .attr('height', 10)
        .attr('fill', "#4126ff") 

      d3.select(`.barchart-svg-${this.chartId}`)
        .select(".plot")
        .append("text")
        .attr("x", 375)
        .attr("y", 10)
        .text(d3.max(this.dataset, (d) => d[this.attribColor]))
        .style("font-size", "10px")
        .attr("alignment-baseline","left")
      d3.select(`.barchart-svg-${this.chartId}`)
        .select(".plot")
        .append("text")
        .attr("x", 305)
        .attr("y", 10)
        .text(d3.min(this.dataset, (d) => d[this.attribColor]))
        .style("font-size", "10px")
        .attr("alignment-baseline","middle")  
      
      var units = "";
      if (this.attribColor=="salary") {
        units = "($)";
      }
      if (this.attribColor=="height") {
        units = "(meters)";
      }
      if (this.attribColor=="weight") {
        units = "(lbs)";
      }
      d3.select(`.barchart-svg-${this.chartId}`)
        .select(".plot")
        .append("text")
        .attr("x", 325)
        .attr("y", -20)
        .text(`${this.attribColor} ${units}`)
        .style("font-size", "10px")
        .attr("alignment-baseline","middle")  
    },
    itemSelected(d) {
        this.$emit('item-selected', d)
    }
  },
  computed: {
    viewBox() {
      return `0 0 ${this.width + this.margin.left + this.margin.right} ${
        this.height + this.margin.top + this.margin.bottom
      }`;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
h1 {
  margin: 0;
  font-size: 16px;
}
</style>