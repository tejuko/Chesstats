<script>
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
</style>