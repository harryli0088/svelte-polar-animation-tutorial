<script lang="ts">
  import { onDestroy } from 'svelte'
  import { tweened } from 'svelte/motion'
  import { cubicOut } from 'svelte/easing'
  import { scaleLinear } from "d3-scale"
  const DEG_PER_RAD:number = 180/Math.PI

  export let fullWidth:number = 500
  $: centeredWidth = Math.min(500, fullWidth)
  $: fullHeight = centeredWidth

  const PADDING = 30
  $: width = centeredWidth - 2*PADDING
  $: height = fullHeight - 2*PADDING
  $: halfWidth = width/2
  $: halfHeight = height/2

  const animation = tweened(0, {
    duration: 4000,
    easing: cubicOut
  })
  const interval = setInterval(() => {
    animation.set($animation===1 ? 0 : 1)
  }, 5000)
  onDestroy(() => clearInterval(interval))

  $: x2Scale = scaleLinear().domain(
    [0, 180, 180, 360]
  ).range(
    [
      0,
      $animation*halfWidth,
      -$animation*halfWidth,
      0,
    ]
  )
  $: y2 = $animation * halfHeight
  $: thetaScale = scaleLinear().domain(
    [0, 180, 180, 360]
  ).range(
    [0, (1 - $animation)*180, ($animation - 1)*180, 0]
  )
  $: getLineDataFromAngle = (angle, radius=halfWidth) => {
    const x2 = x2Scale(angle)
    const theta = thetaScale(angle) / DEG_PER_RAD
    return {
      x1: x2 + radius * Math.sin(theta),
      y1: - radius * Math.cos(theta),
      x2, y2,
    }
  }
  const circleDegrees:number[] = []
  for(let i=180; i<360; ++i) {
    circleDegrees.push(i)
  }
  for(let i=0; i<180; ++i) {
    circleDegrees.push(i)
  }
  circleDegrees.push(179.9) //this is now equal to [180, 181, ..., 359, 0, 1, ..., 179, 179.9]
  $: topFactor = (12 + 3*$animation) / 16
  $: circlePathRadius = halfWidth * topFactor
  $: circlePath = circleDegrees.reduce(
    (d, degree) => {
      const { x1, y1 } = getLineDataFromAngle(degree, circlePathRadius)
      d += ` ${x1},${y1}`
      return d
    },
    "M"
  )
  const ticks:{angle:number,label:string, dy:number}[] = [
    {angle:180,     label: "180°"},
    {angle:225,     label: ""},
    {angle:270,     label: "270°"},
    {angle:315,     label: ""},
    {angle:0,       label: "0° (360°)"},
    {angle:45,      label: ""},
    {angle:90,      label: "90°"},
    {angle:135,     label: ""},
    {angle:179.999, label: "180°"},
  ]
  $: lineData = ticks.map(t => getLineDataFromAngle(t.angle))
  $: ticksWithLabels = ticks.filter(t =>
    t.label.length > 0 //only allow ticks with a label
    && !(Math.floor(t.angle)===179 && $animation===0) //hide the 179.999 label if the animation is at 0
  )

  $: textData = ticksWithLabels.map(t => {
    const { x1, y1 } = getLineDataFromAngle(t.angle)
    return {
      attr: {
        x: x1,
        y: y1,
      },
      label: t.label,
    }
  })
</script>

<main>
  <div bind:clientWidth={fullWidth}>
    <svg
      width={fullWidth}
      height={fullHeight}
    >
      <g transform={`translate(${fullWidth/2},${fullHeight/2})`}>
        <g class="grid">
          <path d={circlePath}/>

          {#each lineData as line}
            <line {...line}/>
          {/each}

          <line x1={x2Scale(180)} y1={y2} x2={x2Scale(179.9)} {y2}/>

          {#each textData as text}
            <text {...text.attr}>{text.label}</text>
          {/each}
        </g>
      </g>
    </svg>
  </div>
</main>

<style>
  .grid line, .grid path {
    fill: none;
    stroke: gray;
    stroke-width: 1px;
  }
  .grid text {
    text-anchor: middle;
  }
  .timeAxis line {
    stroke: gray;
    stroke-width: 2px;
  }
</style>
