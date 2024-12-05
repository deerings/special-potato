<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  onMount(() => {
      const margin = { top: 50, right: 150, bottom: 50, left: 100 };
      const width = 800 - margin.left - margin.right;
      const height = 500 - margin.top - margin.bottom;

      const svg = d3.select('#chart')
          .append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
          .append('g')
          .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = d3.scaleTime().range([0, width]);
      const y = d3.scaleLinear().range([height, 0]);

      const xAxis = svg.append('g').attr('transform', `translate(0,${height})`);
      const yAxis = svg.append('g');

      const line = d3.line()
          .x(d => x(new Date(d.date)))
          .y(d => y(d.value));

      d3.csv('./housing.csv').then(data => {
          // Process data
      });
  });
</script>

<style>
  body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
  }

  .container {
      display: flex;
      flex-direction: column;
      align-items: center;
      text-align: center;
      background-color: white;
      padding: 20px;
      border-radius: 10px;
      width: 80%;
      max-width: 1200px;
  }

  h1 {
      margin-top: 20px;
      margin-bottom: 40px;
  }

  #controls {
      margin-bottom: 20px;
  }

  select {
      margin: 10px;
      padding: 10px;
      font-size: 16px;
      border-radius: 5px;
      border: 1px solid #ccc;
  }

  .legend-box {
      fill: white;
      stroke: black;
      opacity: 0.8;
  }

  .legend-item {
      font-size: 12px;
  }

  #chart {
      width: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
  }
</style>

<div class="container">
  <h1>Housing</h1>
  <p>
      This is an interactive visualization of housing trends in the United States.
      The data is sourced from the Zillow website where the Zillow Home Value Index
      is used to determine the typical value of a home by month.
  </p>
  <div id="controls">
      <select id="dropdown1"></select>
      <select id="dropdown2"></select>
      <select id="dropdown3"></select>
      <select id="dropdown4"></select>
  </div>
  <div id="chart"></div>
</div>
