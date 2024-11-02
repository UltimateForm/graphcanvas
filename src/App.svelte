<script lang="ts">
  import * as d3 from "d3";
  import type { Line, DataObj } from "./types";
  const data = $state<DataObj[]>([]);
  const lines = $state<Line[]>([]);
  let drawing = $state<boolean>(true);
  let height = $state(0);
  let width = $state(0);
  const yScale = $derived(d3.scaleLinear([0, height]).domain([0, 500]));
  const xScale = $derived(d3.scaleLinear().range([0, width]).domain([0, 1000]));
  function getRandomColor() {
    return (
      "#" + (((1 << 24) * Math.random()) | 0).toString(16).padStart(6, "0")
    );
  }
  function handleDrawNext(x: number, y: number, closer: boolean = false) {
    const unfinishedLine = lines.findLast((line) => !line.to);
    if (unfinishedLine) {
      unfinishedLine.to = [x, y];
    }
    if (!closer) {
      data.push({
        r: 15,
        x,
        y,
        f: getRandomColor(),
      });
      lines.push({
        from: [x, y],
      });
    }
  }
  function handleCanvasClick(event: MouseEvent) {
    if (!drawing) return;
    const x = xScale.invert(event.clientX);
    const y = yScale.invert(event.clientY);
    handleDrawNext(x, y);
  }
  function handleCircleClick(index: number) {
    return (event: MouseEvent) => {
      if (index !== 0) return;
      const x = xScale.invert(event.clientX);
      const y = yScale.invert(event.clientY);
      handleDrawNext(x, y, true);
      drawing = false;
    };
  }
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
<main
  bind:clientHeight={height}
  bind:clientWidth={width}
  onclick={handleCanvasClick}
>
  <svg {width} {height}>
    {#each lines?.filter((line) => line.to) ?? [] as { from, to }}
      <line
        x1={xScale(from[0])}
        y1={yScale(from[1])}
        x2={xScale(to![0])}
        y2={yScale(to![1])}
        fill="#ffffff"
        style="stroke:white;stroke-width:2"
      />
    {/each}
    {#each data as { x, y, r, f }, index}
      <circle
        cx={xScale(x)}
        cy={yScale(y)}
        {r}
        fill={f}
        onclick={handleCircleClick(index)}
        role="button"
        tabindex={index}
      />
    {/each}
  </svg>
</main>

<style>
  svg {
    background: #00000e;
  }

  main {
    width: 100%;
    height: 100%;
  }
  circle:first-of-type {
    cursor: pointer;
  }
  circle {
    transition: r 0.5s ease-out;
  }
</style>
