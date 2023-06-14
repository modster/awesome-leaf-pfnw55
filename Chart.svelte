<script>
  import * as d3 from "d3";
  import { findClosestPoint } from "./utils";

  export let allPeople = [];
  export let people = [];
  export let metric1 = "";
  export let metric2 = "";
  let wrapper;

  $: xAccessor = d => d.powerstats[metric1];
  $: yAccessor = d => d.powerstats[metric2];

  const colors = {
    good: "#5fa7b5",
    bad: "#760765"
  };
  const colorAccessor = d => colors[d.biography.alignment];

  let height = 400;
  $: width = height;

  let r = 16;

  $: xScale = d3
    .scaleLinear()
    .domain(d3.extent(allPeople, xAccessor))
    .range([0, width]);

  $: yScale = d3
    .scaleLinear()
    .domain(d3.extent(allPeople, yAccessor))
    .range([height, 0]);

  let peoplePositions = [];
  let simulation = d3.forceSimulation();
  const updatePositions = () => {
    peoplePositions = people.map(d => ({
      ...d,
      x: xScale(xAccessor(d)),
      targetX: xScale(xAccessor(d)),
      y: yScale(yAccessor(d)),
      targetY: yScale(yAccessor(d)),
      color: colorAccessor(d)
    }));
    simulation
      .nodes(peoplePositions)
      .force("x", d3.forceX(d => d.targetX).strength(1))
      .force("y", d3.forceY(d => d.targetY).strength(1))
      .force(
        "collide",
        d3
          .forceCollide(d => 3)
          .iterations(3)
          .strength(0.1)
      )
      .on("tick", d => {
        peoplePositions = peoplePositions;
      });

    simulation.alpha(0.8);
    simulation.restart();
  };
  $: metric1, metric2, width, people, updatePositions();

  let hoveredPointName = null;
  $: hoveredPoint = people.find(d => d.name === hoveredPointName) || {};
  $: console.log(hoveredPoint);
</script>

<div
  class="wrapper"
  bind:this={wrapper}
  bind:clientHeight={height}
  style="width: {width}px"
  on:mousemove={e => {
    if (!wrapper) return
    const bounds = wrapper.getBoundingClientRect()
    const closest = findClosestPoint(
      peoplePositions,
      [e.clientX - bounds.left, e.clientY - bounds.top]
    )
    hoveredPointName = closest && closest.name
  }}
  on:mouseleave={e => hoveredPointName = null}>
  {#each peoplePositions as person}
    <div
      class="image"
      style={`
        background-image: url(${person.images?.sm});
        border-color: ${
           person.color
          //hoveredPointName === person.name ? "black" : "transparent"
        };
        z-index: ${hoveredPointName === person.name ? 10 : 1};
        height: ${r*2}px;
        width: ${r*2}px;
        transform: translate(calc(-50% + ${person.x}px), calc(-50% + ${person.y}px)) scale(${hoveredPointName === person.name ? 1.5 : 1});
      `}
    />
  {/each}

  {#if hoveredPointName && hoveredPoint}
    <div class="tooltip">
      {hoveredPointName}
      <img src={hoveredPoint.images?.sm} alt={hoveredPointName} />
    </div>
  {/if}

  <div class="vertical-line">⇡ More {metric2}</div>
  <div class="horizontal-line">More {metric1} ⇢</div>
</div>

<style>
  .wrapper {
    position: relative;
    display: flex;
    flex-direction: column;
    height: 80vh;
    margin: 1em auto;
  }
  .image {
    position: absolute;
    background-size: cover;
    border-radius: 100%;
    border: 3px solid transparent;
    overflow: hidden;
    transition: transform 0.3s ease-out;
  }
  .tooltip {
    position: absolute;
    top: 0;
    left: 0;
    font-size: 0.8em;
    transform: translateY(-50%);
    pointer-events: none;
  }
  .tooltip img {
    display: block;
    width: 4em;
  }
  .vertical-line {
    position: absolute;
    left: 50%;
    top: 0;
    bottom: 0;
    width: 0;
    border-right: 1px solid black;
    display: flex;
    align-items: flex-start;
    text-align: left;
    font-size: 0.7em;
    opacity: 0.6;
    font-style: italic;
    font-weight: 300;
  }
  .horizontal-line {
    position: absolute;
    top: 50%;
    left: 0;
    right: 0;
    border-bottom: 1px solid black;
    height: 0;
    display: flex;
    justify-content: flex-end;
    text-align: left;
    font-size: 0.7em;
    opacity: 0.6;
    font-style: italic;
    font-weight: 300;
  }
</style>
