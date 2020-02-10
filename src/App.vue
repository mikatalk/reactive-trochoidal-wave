<template>
  <svg id="app" xmlns="http://www.w3.org/2000/svg"
    :width="width" 
    :height="height"
    preserveAspectRatio="xMidYMid meet"
    :view-box.camel="`0 0 ${width} ${height}`"
  >
    <g 
      v-for="(line, lineIndex) in lines"
      :key="lineIndex"
      :transform="`translate(0 ${lineIndex * 20 - lines.length * 10 / 2})`"
    >
      <circle 
        v-for="(point, index) in line.filter((row, index) => index + 2 < line.length)"
        :key="`dot-${index}`"
        :cx="point.x"
        :cy="point.y"
        :r="point.radius" stroke="rgba(255,255,255,0.25)" stroke-width="1" fill="transparent" />
      <polyline
        :opacity="lineIndex / lines.length / 2 + 0.25"
        :points="pointsToString(lineIndex)" stroke="rgba(255,255,255,1)"  stroke-width="0.1" fill="#00BFFF" />
      <circle 
        v-for="(point, index) in animatedLines[lineIndex].filter((row, index) => index > 0 && index + 1 < line.length)" 
        :key="`circle-${index}`"
        :cx="point.x"
        :cy="point.y"
        :r="2" stroke="rgba(255,255,255,0.5)" stroke-width="1" fill="#00BFFF" />
    </g>
  </svg>
</template>

<script>
export default {
  name: 'App',
  data: () => ({
    lines: [],
    width: window.innerWidth,
    height: window.innerHeight,
    resolution: 8,
    time: 0
  }),
  created () {
    this.lines = []
    this.circles = []
    for (let lineIndex = 0, numLines = 3; lineIndex < numLines; lineIndex += 1) {
      this.lines.push([])
      for (let pointIndex = 0, X = this.resolution + 1; pointIndex < X; pointIndex += 1) {
        const radius = window.innerWidth / 20 * (0.2 + (1 -lineIndex / numLines) * 0.8)
        const scale = this.width / this.resolution
        const x = (pointIndex + 1) * scale
        const y = this.height / 2
        this.lines[lineIndex].push({x, y, radius})
      }
    }
    this.loop(0)
  },
  methods: {
    loop (time) {
      window.requestAnimationFrame(this.loop)
      this.time = time * 0.003
    },
    pointsToString (lineIndex) {
      const points = []
      const lines = this.animatedLines
      // bottom left fixed point
      points.push({x: 0, y: this.height * 2})
      // left moving point
      points.push({x: 0, y: lines[lineIndex][0].y})
      points.push(...lines[lineIndex])
      // right moving point
      points.push({x: this.width, y: lines[lineIndex][lines[lineIndex].length - 1].y})
      // bottom right fixed point
      points.push({x: this.width, y: this.height * 2})
      return points.map(pt => `${pt.x},${pt.y}`)
    }
  },
  computed: {
    animatedLines () {
      const step = this.width / this.resolution
      return this.lines.map(line => line.map((point, pointIndex) => ({
        x: step * pointIndex + Math.sin(this.time + pointIndex) * point.radius,
        y: this.height / 2 + Math.cos(this.time + pointIndex) * point.radius
      })))
    },
    viewBox () {
      return `0 0 ${this.width} ${this.height}`
    },
    
  }
}
</script>

<style lang="scss">
body {
  background: black;
  margin: 0;
  overflow: hidden;
  svg#app {
    border: 0px solid black;
    width: 100vw;
    box-sizing: border-box;
    overflow: hidden;
  }
}
</style>
