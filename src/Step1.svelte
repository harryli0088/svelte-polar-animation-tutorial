<script lang="ts">
  export let fullWidth:number = 500
  $: centeredWidth = Math.min(500, fullWidth)
  $: fullHeight = centeredWidth
  const PADDING = 30
  $: width = centeredWidth - 2*PADDING
  $: height = fullHeight - 2*PADDING
  $: halfWidth = width/2
  $: halfHeight = height/2

  $: ticks = [ //hard code the tick positions
    {angle:180,     label: "180°",      x1: 0,                       y1: halfHeight,               x2: 0, y2: 0},
    {angle:225,     label: "",          x1: -halfWidth/Math.sqrt(2), y1: -halfHeight/Math.sqrt(2), x2: 0, y2: 0},
    {angle:270,     label: "270°",      x1: -halfWidth,              y1: 0,                        x2: 0, y2: 0},
    {angle:315,     label: "",          x1: -halfWidth/Math.sqrt(2), y1: halfHeight/Math.sqrt(2),  x2: 0, y2: 0},
    {angle:0,       label: "0° (360°)", x1: 0,                       y1: -halfHeight,              x2: 0, y2: 0},
    {angle:45,      label: "",          x1: halfWidth/Math.sqrt(2),  y1: -halfHeight/Math.sqrt(2), x2: 0, y2: 0},
    {angle:90,      label: "90°",       x1: halfWidth,               y1: 0,                        x2: 0, y2: 0},
    {angle:135,     label: "",          x1: halfWidth/Math.sqrt(2),  y1: halfHeight/Math.sqrt(2),  x2: 0, y2: 0},
    {angle:179.999, label: "",          x1: 0,                       y1: halfHeight,               x2: 0, y2: 0},
  ]

  $: lineData = ticks.map(t => ({ x1: t.x1, y1: t.y1, x2: t.x2, y2: t.y2 }))
  $: textData = ticks.map(t => ({
    attr: {x: t.x1, y: t.y1},
    label: t.label,
  }))
</script>

<main>
  <div bind:clientWidth={fullWidth}>
    <svg
      width={fullWidth}
      height={fullHeight}
    >
      <g transform={`translate(${fullWidth/2},${fullHeight/2})`}>
        <g class="grid">
          <circle cx={0} cy={0} r={halfWidth*3/4}/>

          {#each lineData as line}
            <line {...line}/>
          {/each}

          {#each textData as text}
            <text {...text.attr}>{text.label}</text>
          {/each}
        </g>
      </g>
    </svg>
  </div>
</main>

<style>
  circle {
    fill: none;
    stroke: gray;
    stroke-width: 1px;
  }

  .grid line, .grid path {
    fill: none;
    stroke: gray;
    stroke-width: 1px;
  }
  .grid text {
    text-anchor: middle;
  }
</style>
