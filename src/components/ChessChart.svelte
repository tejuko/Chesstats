<script>
  import { onMount, tick } from 'svelte'; // Lifecycle hook & tick for waiting DOM updates
  import * as d3 from 'd3';              // D3.js for chart visualization

  export let username;                   // Chess.com username to fetch data for
  export let year = new Date().getFullYear(); // Default to current year

  let chartContainer;  // DOM element to render D3 chart into
  let error = "";      // Error message
  let loading = true;  // Loading indicator

  // Monthly statistics arrays
  let monthlyWins = [];
  let monthlyDraws = [];
  let monthlyLosses = [];

  // Filters to toggle data series visibility
  let showWins = true;
  let showDraws = true;
  let showLosses = true;

  const keys = ["wins", "draws", "losses"]; // Stack keys for D3
  const colors = {
    wins: "#27ae60",  // Green for wins
    draws: "#fbd043", // Yellow for draws
    losses: "#c0392b" // Red for losses
  };

  // Fetch games for a specific month
  async function fetchMonthGames(year, month) {
    const mm = month.toString().padStart(2, "0");
    const url = `https://api.chess.com/pub/player/${username.toLowerCase()}/games/${year}/${mm}`;

    try {
      const res = await fetch(url);
      if (res.status === 404) return [];      // No games for month
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      return data.games || [];
    } catch (err) {
      console.error("Month fetch error", err);
      return [];
    }
  }

  // Build monthly statistics and render chart
  async function buildChart() {
    try {
      if (!username) {
        error = "Missing username";
        return;
      }

      await tick(); // Wait for DOM to update
      if (!chartContainer) {
        setTimeout(buildChart, 100); // Retry if chart container not ready
        return;
      }

      // Reset monthly stats
      monthlyWins = [];
      monthlyDraws = [];
      monthlyLosses = [];

      // Loop through each month
      for (let month = 1; month <= 12; month++) {
        const games = await fetchMonthGames(year, month);
        let wins = 0, draws = 0, losses = 0;

        // Count results for this month
        for (const g of games) {
          const white = g.white?.username?.toLowerCase();
          const black = g.black?.username?.toLowerCase();
          const resultW = g.white?.result || "";
          const resultB = g.black?.result || "";

          if (white === username.toLowerCase()) {
            if (resultW === "win") wins++;
            else if (["agreed","repetition","stalemate","timevsinsufficient"].includes(resultW)) draws++;
            else losses++;
          } else if (black === username.toLowerCase()) {
            if (resultB === "win") wins++;
            else if (["agreed","repetition","stalemate","timevsinsufficient"].includes(resultB)) draws++;
            else losses++;
          }
        }

        monthlyWins.push(wins);
        monthlyDraws.push(draws);
        monthlyLosses.push(losses);
      }

      // Draw chart with filtered data
      drawD3(
        showWins ? monthlyWins : monthlyWins.map(() => 0),
        showDraws ? monthlyDraws : monthlyDraws.map(() => 0),
        showLosses ? monthlyLosses : monthlyLosses.map(() => 0)
      );

    } catch (err) {
      error = `Failed to build chart: ${err.message}`;
      console.error(err);
    } finally {
      loading = false;
    }
  }

  // Redraw chart when filters change
  $: if (!loading && !error) {
    drawD3(
      showWins ? monthlyWins : monthlyWins.map(() => 0),
      showDraws ? monthlyDraws : monthlyDraws.map(() => 0),
      showLosses ? monthlyLosses : monthlyLosses.map(() => 0)
    );
  }

  // D3 chart rendering function
  function drawD3(wins, draws, losses) {
    if (!chartContainer) return;
    chartContainer.innerHTML = ""; // Clear previous chart

    // Prepare stacked data
    const data = d3.range(12).map((i) => ({
      month: ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"][i],
      wins: wins[i],
      draws: draws[i],
      losses: losses[i]
    }));

    const stack = d3.stack().keys(keys);
    const stackedData = stack(data);

    const width = chartContainer.clientWidth;
    const height = 350;
    const margin = { top: 40, right: 20, bottom: 40, left: 40 };

    const svg = d3.select(chartContainer)
      .append("svg")
      .attr("width", width)
      .attr("height", height);

    const chartWidth = width - margin.left - margin.right;
    const chartHeight = height - margin.top - margin.bottom;

    const g = svg.append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    const x = d3.scaleBand()
      .domain(data.map(d => d.month))
      .range([0, chartWidth])
      .padding(0.2);

    const y = d3.scaleLinear()
      .domain([0, d3.max(stackedData[stackedData.length - 1], d => d[1])])
      .nice()
      .range([chartHeight, 0]);

    // Tooltip
    const tooltip = d3.select(chartContainer)
      .append("div")
      .attr("class", "tooltip")
      .style("position", "absolute")
      .style("pointer-events", "none")
      .style("opacity", 0)
      .style("background", "#333")
      .style("color", "#fff")
      .style("padding", "6px 10px")
      .style("border-radius", "4px")
      .style("font-size", "12px");

    // Draw stacked bars
    g.selectAll("g.series")
      .data(stackedData)
      .enter()
      .append("g")
      .attr("fill", d => colors[d.key])
      .selectAll("rect")
      .data(d => d.map(datum => ({ ...datum, key: d.key })))
      .enter()
      .append("rect")
      .attr("x", d => x(d.data.month))
      .attr("y", y(0))
      .attr("height", 0)
      .attr("width", x.bandwidth())
      .on("mouseover", (event, d) => {
        tooltip.transition().duration(200).style("opacity", 0.9);
        tooltip.html(
          `<strong>${d.data.month}</strong><br/>
          Wins: ${d.data.wins}<br/>
          Draws: ${d.data.draws}<br/>
          Losses: ${d.data.losses}`
        );
        const xPos = x(d.data.month) + x.bandwidth() / 2;
        const yPos = y(d[1]);
        tooltip
          .style("left", xPos + "px")
          .style("top", (yPos - 10) + "px")
          .style("transform", "translateX(-50%)");
      })
      .on("mousemove", (event, d) => {
        const xPos = x(d.data.month) + x.bandwidth() / 2;
        const yPos = y(d[1]);
        tooltip
          .style("left", xPos + "px")
          .style("top", (yPos - 10) + "px")
          .style("transform", "translateX(-50%)");
      })
      .on("mouseout", () => {
        tooltip.transition().duration(200).style("opacity", 0);
      })
      .transition()
      .duration(700)
      .attr("y", d => y(d[1]))
      .attr("height", d => y(d[0]) - y(d[1]));

    // X and Y axes
    g.append("g")
      .attr("transform", `translate(0,${chartHeight})`)
      .call(d3.axisBottom(x));

    g.append("g").call(d3.axisLeft(y));

    // Chart title
    svg.append("text")
      .attr("x", width / 2)
      .attr("y", 20)
      .attr("text-anchor", "middle")
      .attr("font-size", "16px")
      .attr("font-weight", "bold")
      .text(`Games Played by ${username} in ${year}`);
  }

  onMount(buildChart); // Initialize chart on mount
</script>

<h2>Your monthly chess stats per year</h2>

{#if loading}
  <p>Loading chart data...</p>
{:else if error}
  <p style="color: red;">{error}</p>
{:else}
  <!-- Filter buttons to toggle Wins/Draws/Losses -->
  <div class="chart-filters">
    <button style="background-color: #27ae60;" on:click={() => showWins = !showWins}>{showWins ? 'Hide' : 'Show'} Wins</button>
    <button style="background-color: #fbd043;" on:click={() => showDraws = !showDraws}>{showDraws ? 'Hide' : 'Show'} Draws</button>
    <button style="background-color: #c0392b; color:white;" on:click={() => showLosses = !showLosses}>{showLosses ? 'Hide' : 'Show'} Losses</button>
  </div>

  <!-- Chart container -->
  <div bind:this={chartContainer} class="chart-wrapper"></div>
{/if}

<style>
  .chart-wrapper {
    width: 100%;
    height: 100%;
    min-height: 350px;
    display: block;
    position: relative;
  }

  .chart-filters {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
    justify-content: center;
  }

  .chart-filters button {
    border: none;
    padding: 6px 12px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: bold;
  }

  .chart-filters button:hover {
    opacity: 0.85;
  }
</style>


<!-- <script>
  import { onMount, tick } from 'svelte';
  import Chart from 'chart.js/auto';

  export let username;
  export let year = new Date().getFullYear();

  let canvasEl;
  let chart;
  let error = '';
  let loading = true;

  // --- Fetch helper --------------------------------------------------
  async function fetchMonthGames(year, month) {
    const mm = month.toString().padStart(2, '0');
    const url = `https://api.chess.com/pub/player/${username.toLowerCase()}/games/${year}/${mm}`;
    try {
      const res = await fetch(url);
      console.log('Fetching:', url, 'Status:', res.status);
      if (res.status === 404) return [];
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      return data.games || [];
    } catch (err) {
      console.error(`Error fetching month ${mm}:`, err);
      return [];
    }
  }

  // --- Chart builder -------------------------------------------------
  async function buildChart() {
    try {
      if (!username) {
        error = 'Missing username';
        return;
      }

      await tick();
      if (!canvasEl) {
        console.warn('Canvas not ready yet, retrying...');
        setTimeout(buildChart, 200);
        return;
      }

      const monthlyWins = [];
      const monthlyLosses = [];
      const monthlyDraws = [];

      // Fetch stats for all 12 months
      for (let month = 1; month <= 12; month++) {
        const games = await fetchMonthGames(year, month);

        let wins = 0, losses = 0, draws = 0;
        for (const game of games) {
          const white = game.white?.username?.toLowerCase();
          const black = game.black?.username?.toLowerCase();
          const result = game.white?.result || '';

          // Determine outcome from perspective of user
          if (white === username.toLowerCase()) {
            if (result === 'win') wins++;
            else if (result === 'agreed' || result === 'repetition' || result === 'stalemate' || result === 'timevsinsufficient') draws++;
            else losses++;
          } else if (black === username.toLowerCase()) {
            const blackResult = game.black?.result || '';
            if (blackResult === 'win') wins++;
            else if (blackResult === 'agreed' || blackResult === 'repetition' || blackResult === 'stalemate' || blackResult === 'timevsinsufficient') draws++;
            else losses++;
          }
        }

        monthlyWins.push(wins);
        monthlyLosses.push(losses);
        monthlyDraws.push(draws);
      }

      const ctx = canvasEl.getContext('2d');
      if (!ctx) throw new Error('Canvas context unavailable');
      if (chart) chart.destroy();

      chart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: [
            'Jan','Feb','Mar','Apr','May','Jun',
            'Jul','Aug','Sep','Oct','Nov','Dec'
          ],
          datasets: [
            {
              label: 'Wins',
              data: monthlyWins,
              backgroundColor: '#27ae60'
            },
            {
              label: 'Draws',
              data: monthlyDraws,
              backgroundColor: '#fbd043'
            },
            {
              label: 'Losses',
              data: monthlyLosses,
              backgroundColor: '#c0392b'
            }
          ]
        },
        options: {
          responsive: true,
          interaction: { mode: 'index', intersect: false },
          plugins: {
            title: {
              display: true,
              text: `Games Played by ${username} in ${year}`,
              color: 'black',
              font: { size: 15 }
            },
            tooltip: {
              callbacks: {
                label: function(context) {
                  return `${context.dataset.label}: ${context.formattedValue}`;
                }
              }
            }
          },
          scales: {
            x: { stacked: true },
            y: { stacked: true, beginAtZero: true, ticks: { stepSize: 1 } }
          }
        }
      });
    } catch (err) {
      console.error('Error building chart:', err);
      error = `Failed to load chart data for ${username}: ${err.message}`;
    } finally {
      loading = false;
    }
  }

  onMount(buildChart);
</script>

<h2>Your Monthly Chess Stats</h2>

{#if loading}
  <p>Loading chart data...</p>
{:else if error}
  <p style="color:red">{error}</p>
{:else}
  <canvas bind:this={canvasEl} width="800" height="400"></canvas>
{/if}

<style>
  canvas {
    max-width: 100%;
    height: auto;
  }
</style> -->