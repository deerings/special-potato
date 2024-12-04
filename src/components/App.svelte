<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Housing Data Visualization</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
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
            text-align: center;
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
            font-family: Arial, sans-serif;
        }

        #chart {
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
        }
    </style>
</head>
<body>

<div class="container">

    <h1>Housing</h1>
    <p>This is an interactive visualization of housing trends in the United States. 
    The data is sourced from the Zillow website where the Zillow Home Value Index 
    is used to determine the typical value of a home by month. The dataset dates back 
    24 years and is separated by state. The dropdown menu can be used to compare the 
    housing trends between four states. </p>

    <div id="controls">
        <select id="dropdown1"></select>
        <select id="dropdown2"></select>
        <select id="dropdown3"></select>
        <select id="dropdown4"></select>
    </div>

    <!-- Chart Container -->
    <div id="chart"></div>
</div>

<script>
    // Dimensions with increased left margin
    const margin = { top: 50, right: 150, bottom: 50, left: 100 }; // Increased left margin
    const width = 800 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;

    // SVG Container
    const svg = d3.select("#chart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

    // Scales
    const x = d3.scaleTime().range([0, width]);
    const y = d3.scaleLinear().range([height, 0]);

    // Axes
    const xAxis = svg.append("g")
        .attr("transform", `translate(0,${height})`);
    const yAxis = svg.append("g");

    // Line generator
    const line = d3.line()
        .x(d => x(new Date(d.date)))
        .y(d => y(d.value));

    // Axis labels
    svg.append("text")
        .attr("text-anchor", "middle")
        .attr("transform", `translate(${width / 2},${height + 40})`)
        .text("Date");

    // Adjusted Y-axis title position
    svg.append("text")
        .attr("text-anchor", "middle")
        .attr("transform", `translate(${-60},${height / 2}) rotate(-90)`) // Moved further left
        .text("Housing Value ($)");

    // Load data
    d3.csv('special-potato/housing.csv').then(data => {
      
        const states = data.columns.slice(1);
        const allStateData = states.map(state => {
            return {
                state: state,
                values: data.map(d => ({ date: d.RegionName, value: +d[state] }))
            };
        });

        // Set dropdowns
        const dropdownIds = ["#dropdown1", "#dropdown2", "#dropdown3", "#dropdown4"];
        dropdownIds.forEach(id => {
            const dropdown = d3.select(id);
            dropdown.append("option").attr("value", "").text("Select a state");
            states.forEach(state => {
                dropdown.append("option").attr("value", state).text(state);
            });
        });

        // Set domains
        x.domain(d3.extent(data, d => new Date(d.RegionName)));
        y.domain([0, d3.max(allStateData, d => d3.max(d.values, v => v.value))]).nice();

        // Draw axes
        xAxis.call(d3.axisBottom(x));
        yAxis.call(d3.axisLeft(y));

        // Update function
        function update() {
            const selectedStates = dropdownIds
                .map(id => document.querySelector(id).value)
                .filter(state => state !== "");

            const selectedData = allStateData.filter(d => selectedStates.includes(d.state));

            // Update the y-axis domain
            y.domain([0, d3.max(selectedData, d => d3.max(d.values, v => v.value))]).nice();
            yAxis.transition().duration(1000).call(d3.axisLeft(y));

            // Update the lines
            const lines = svg.selectAll(".state-line")
                .data(selectedData, d => d.state);

            lines.enter()
                .append("path")
                .attr("class", "state-line")
                .merge(lines)
                .transition()
                .duration(1000)
                .attr("d", d => line(d.values))
                .attr("stroke", (_, i) => d3.schemeCategory10[i % 10])
                .attr("fill", "none");

            lines.exit().remove();

            // Legend Group
            const legendGroup = svg.selectAll(".legend-group")
                .data([null]);

            const legendEnter = legendGroup.enter()
                .append("g")
                .attr("class", "legend-group");

            legendEnter.merge(legendGroup)
                .attr("transform", `translate(${width + 20}, 20)`);

            // Legend Box
            legendEnter.append("rect")
                .attr("class", "legend-box")
                .merge(legendGroup.select(".legend-box"))
                .attr("width", 130)
                .attr("height", selectedData.length * 20 + 10)
                .attr("x", -10)
                .attr("y", -10);

            // Legend Items
            const legendItems = legendGroup.selectAll(".legend-item")
                .data(selectedData, d => d.state);

            legendItems.enter()
                .append("text")
                .attr("class", "legend-item")
                .merge(legendItems)
                .attr("x", 0)
                .attr("y", (_, i) => i * 20)
                .attr("fill", (_, i) => d3.schemeCategory10[i % 10])
                .text(d => d.state);

            legendItems.exit().remove();

            legendGroup.exit().remove();
        }

        // Add event listeners
        dropdownIds.forEach(id => {
            d3.select(id).on("change", update);
        });

        update();
    });
</script>


</body>
</html>

