<template id="sunburst">
    <section class="graph">

    </section>
</template>


<script>
export default {
  name: 'sunburst',
  props: ['chartData'],
  data: function() {
    return {};
  },
  mounted: function() {
    /* eslint-disable */
    var width = document.querySelector('.graph').clientWidth;
    var height = document.querySelector('.graph').clientHeight;
    var radius = Math.min(width, height) / 2;
    var color = d3.scale.category20c();
    var formatNumber = d3.format(",d");
    var x = d3.scale.linear().range([0, 2 * Math.PI]);
    var y = d3.scale.sqrt().range([0, radius]);
    var partition = d3.layout.partition().value(function(d) { return d.size; });
    var arc = d3.svg.arc()
      .startAngle(function(d) { return Math.max(0, Math.min(2 * Math.PI, x(d.x))); })
      .endAngle(function(d) { return Math.max(0, Math.min(2 * Math.PI, x(d.x + d.dx))); })
      .innerRadius(function(d) { return Math.max(0, y(d.y)); })
      .outerRadius(function(d) { return Math.max(0, y(d.y + d.dy)); });
    var svg = d3.select(".graph").append("svg")
      .attr("width", width)
      .attr("height", height)
    .append("g")
      .attr("transform", "translate(" + width / 2 + "," + (height / 2) + ")");

    d3.json(this.chartData, function(error, root) {
      if (error) throw error;

      svg.selectAll("path")
          .data(partition.nodes(root))
        .enter().append("path")
          .attr("d", arc)
          .attr("stroke-width", function (d) {
            return 2;
          })
          .attr("stroke", function (d) {
            return '#FFF';
          })
          .style("fill", function(d) { return color((d.children ? d : d.parent).name); })
          .on("click", click)
        .append("title")
          .text(function(d) { return d.name + "\n" + formatNumber(d.value); });
    });

    function click(d) {
      svg.transition()
          .duration(750)
          .tween("scale", function() {
            var xd = d3.interpolate(x.domain(), [d.x, d.x + d.dx]),
                yd = d3.interpolate(y.domain(), [d.y, 1]),
                yr = d3.interpolate(y.range(), [d.y ? 20 : 0, radius]);
            return function(t) { x.domain(xd(t)); y.domain(yd(t)).range(yr(t)); };
          })
        .selectAll("path")
          .attrTween("d", function(d) { return function() { return arc(d); }; });
    }

    d3.select(self.frameElement).style("height", height + "px");
  }
};

</script>

<style>
.graph {
  margin: auto;
  width: 100%;
  min-height: 800px;
}
</style>
