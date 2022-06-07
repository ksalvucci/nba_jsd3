
<template>
  <div>
    <h1>{{title}}</h1>
    <svg :class="`scatterplot-svg-${chartId}`" :viewBox="viewBox">

        <!-- Chart and Axes -->
        <g :transform="`translate(${margin.left}, ${margin.top})`">
            <g class="plot"></g>
            <g class="x-axis" :transform="`translate(0, ${height})`"></g>
            <g class="y-axis"></g>
        </g>

        <!-- Axis Labels -->
        <g :transform="`translate(${margin.left + (width/3)}, ${margin.top + margin.bottom/1.5 + height})`">
            <text>{{labelX}}</text>
        </g>
        <g :transform="`translate(${margin.left/4}, ${margin.top + (height)}) rotate(-90)`">
            <text>{{labelY}}</text>
        </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "ScatterPlot",
  props: {
      chartId: {
          required: true,
          type: String
      },
      title: {
          type: String
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
    labelY: {
        type: String
    },
    attribColor: {
      required: false,
      type: String,
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
          top: 50,
          right: 20,
          bottom: 75,
      }
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      console.log(d3); // This can be savely removed
      // This will be called when the page loads.
      // You can load your data and setup D3 here
      this.x = d3.scaleLinear();
      this.y = d3.scaleLinear();
      this.color = d3.scaleOrdinal();

      this.update();
    },
    update() {
      this.x.domain(d3.extent(this.dataset, d => d[this.attribX])).range([0, this.width])
      this.y.domain(d3.extent(this.dataset, d => d[this.attribY])).range([this.height, 0])
      this.color.domain(this.dataset, d => d[this.attribColor]).range(d3.schemeTableau10)

      this.renderScatter();
      this.renderXAxis();
      this.renderYAxis();
    },
renderScatter() {
      // Plot normal points
      d3.select(`.scatterplot-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("circle")
        .data(this.dataset)
        .join("circle")
        .transition()
        .duration(200)
        .attr("cx", (d) => this.x(d[this.attribX]))
        .attr("cy", (d) => this.y(d[this.attribY]))
        .attr("fill", (d) => this.color(d[this.attribColor]))

      d3.select(`.scatterplot-svg-${this.chartId}`)
        .select(".plot")
        .selectAll("circle")
        .on('click', (event, d) => {
            this.itemSelected(d)
        })
        .on('mouseover', function(d, i) {
            d3.select(this).attr('class', 'hover');

          let newX = parseFloat(d3.select(this).attr("cx"));
          let newY = parseFloat(d3.select(this).attr("cy")) + 30;

          let name = i.name == null ? "Free Agents" : i.name;

          tooltip
            .attr("x", newX)
            .attr("y", newY)
            .text(name)
            .transition()
            .duration(200)
            .style("opacity", 1);
        })
        .on('mouseout', function() {
            d3.select(this).attr('class', null);
            tooltip.transition().duration(200).style("opacity", 0);
        });

      var g = d3.select(`.scatterplot-svg-${this.chartId}`).append("g");
      var tooltip = g
        .append("text")
        .attr("class", "tooltip")
        .style("opacity", 0);
    },
    renderXAxis() {
        let xAxis = d3.axisBottom(this.x)

        d3.select(`.scatterplot-svg-${this.chartId}`)
          .select('.x-axis')
          .call(xAxis)
    },
    renderYAxis() {
        let yAxis = d3.axisLeft(this.y)

        d3.select(`.scatterplot-svg-${this.chartId}`)
          .select('.y-axis')
          .call(yAxis)
    },
    itemSelected(d) {
        console.log(`Selected ${d}`)
        this.$emit('item-selected', d)
    }
  },
  computed: {
    viewBox() {
      return `0 0 ${this.width+this.margin.left+this.margin.right} ${this.height+this.margin.top+this.margin.bottom}`
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.plot rect {
    fill-opacity: 0.5;
}

.plot rect.hover {
    fill-opacity: 1;
    stroke: black;
}

.plot circle {
    fill-opacity: 0.5;
    r: 4;
}

.plot circle.hover {
    fill-opacity: 1;
    r: 6;
    stroke: black;
}
</style>