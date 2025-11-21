<script>
  // Props received from parent component
  export let playerData;
  export let loading;
  export let error;
</script>

<div class="dashboard-card">
    <h2>Your personal chess dashboard</h2>

    {#if loading}
        <!-- Loading state -->
        <p class="loading">Loading player data...</p>

    {:else if error}
        <!-- Error state -->
        <p class="error">{error}</p>

    {:else if playerData}
        <!-- Ratings section -->
        <div class="ratings">
            <h3>Your Ratings</h3>

            <!-- Three rating types rendered dynamically -->
            <div class="rating-grid">
                {#each 
                    [
                        { 
                            type: 'rapid', 
                            icon: '⏳', 
                            data: playerData.chess_rapid, 
                            tooltip: 'Rapid: longer time controls (10–15 minutes, often with increments).' 
                        },
                        { 
                            type: 'blitz', 
                            icon: '⚡', 
                            data: playerData.chess_blitz, 
                            tooltip: 'Blitz: fast-paced games (3+2, 5 minutes, etc).' 
                        },
                        { 
                            type: 'bullet', 
                            icon: '🔫', 
                            data: playerData.chess_bullet, 
                            tooltip: 'Bullet: extremely fast games, usually 1 minute.' 
                        }
                    ] as rating}
                    
                    <!-- Individual rating box -->
                    <div class="rating-item {rating.type}">
                        <div class="tooltip">

                            <!-- Tooltip text shown on hover -->
                            <span class="tooltiptext {rating.type}">
                                {rating.tooltip}
                            </span>

                            <div class="rating-header">
                                <span class="icon">{rating.icon}</span>

                                <!-- Capitalize first letter of rating type -->
                                <h4>{rating.type.charAt(0).toUpperCase() + rating.type.slice(1)}</h4>
                            </div>

                            <!-- Rating value (or N/A if not available) -->
                            <p class="rating-value">
                                {rating.data?.last?.rating ?? 'N/A'}
                            </p>

                            <!-- Win / Loss / Draw display -->
                            <div class="record">
                                <div class="stat win">
                                    <span class="emoji">🏆</span>
                                    <span class="value">{rating.data?.record?.win ?? 0}</span>
                                </div>

                                <div class="stat loss">
                                    <span class="emoji">💀</span>
                                    <span class="value">{rating.data?.record?.loss ?? 0}</span>
                                </div>

                                <div class="stat draw">
                                    <span class="emoji">🤝</span>
                                    <span class="value">{rating.data?.record?.draw ?? 0}</span>
                                </div>
                            </div>

                        </div>    
                    </div>
                {/each}
            </div>
        </div>
    {/if}

    <!-- Legend for icons -->
    <div class="legenda">
        <p>🏆 = Win</p>
        <p>💀 = Loss</p>
        <p>🤝 = Draw</p>
    </div>
</div>

<style>
  /* Title */
  h2 {
    color: black;
  }

  /* Loading and error text */
  .loading, .error {
    color: #666;
    font-weight: 500;
  }
  .error {
    color: #e74c3c;
  }

  /* Ratings container */
  .ratings {
    text-align: left;
  }
  .ratings h3 {
    color: black;
    margin-bottom: 1rem;
    border-bottom: 2px solid #eee;
    padding-bottom: 0.3rem;
  }

  /* Layout for the rating boxes */
  .rating-grid {
    display: flex;
    gap: 1rem;
  }

  /* Each rating box */
  .rating-item {
    flex: 1;
    border-left: 5px solid #4e7837; /* default border */
    border-radius: 14px;
    padding: 1.2rem;
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.06);
    transition: all 0.25s ease;
    position: relative;
  }

  /* Custom border colors per category */
  .rating-item.rapid { border-left: 5px solid #27ae60; }
  .rating-item.blitz { border-left: 5px solid #f1c40f; }
  .rating-item.bullet { border-left: 5px solid #e74c3c; }

  /* Hover effect */
  .rating-item:hover {
    transform: translateY(-4px);
    background: linear-gradient(145deg, #fff 0%, #f5f5f5 100%);
    box-shadow: 0 5px 12px rgba(0, 0, 0, 0.1);
  }

  /* Header inside rating box */
  .rating-header {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  .icon {
    font-size: 1.4rem;
  }
  .rating-header h4 {
    font-size: 1.1rem;
    font-weight: 600;
    margin: 0;
    text-transform: capitalize;
  }

  /* Main rating number */
  .rating-value {
    font-size: 1.8rem;
    font-weight: 700;
    margin: 0.5rem 0 0.6rem;
    color: #111;
  }

  /* Win/loss/draw layout */
  .record {
    display: flex;
    justify-content: space-between;
    gap: 0.4rem;
  }
  .stat {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-weight: 600;
    padding: 0.6rem 0;
    border-radius: 8px;
    color: white;
    transition: transform 0.2s ease, filter 0.2s ease;
  }

  /* Emoji & number */
  .stat .emoji {
    font-size: 1.2rem;
  }
  .stat .value {
    font-size: 0.85rem;
    margin-top: 0.2rem;
  }

  /* Colors for win/loss/draw categories */
  .stat.win  { background-color: #27ae60; }
  .stat.loss { background-color: #c0392b; }
  .stat.draw { background-color: #8d8b7f; }

  /* Hover scaling effect */
  .stat:hover {
    transform: scale(1.05);
    filter: brightness(1.1);
  }

  /* Legend at bottom */
  .legenda {
    background: white;
    border-radius: 16px;
    box-shadow: 0 6px 15px rgba(0, 0, 0, 0.08);
    margin-top: 1rem;
    font-size: 0.9rem;
    color: black;
    display: flex;
    gap: 1.5rem;
    justify-content: center;
    padding: 0.5rem 0;
  }
  .legenda p {
    font-weight: 600;
  }

  /* Tooltip wrapper */
  .tooltip {
      position: relative;
      cursor: pointer;
  }

  /* Hidden tooltip bubble */
  .tooltiptext {
      visibility: hidden;
      width: 100%;
      color: #fff;
      text-align: center;
      padding: 5px 0;
      border-radius: 6px;
      position: absolute;
      z-index: 1;
      bottom: 125%;
      left: 50%;
      transform: translateX(-50%);
      opacity: 0;
      transition: opacity 0.3s;
  }

  /* On hover, show tooltip */
  .tooltip:hover .tooltiptext {
      visibility: visible;
      opacity: 1;
  }

  /* Tooltip colors */
  .tooltiptext.rapid { background-color: #27ae60; }
  .tooltiptext.blitz { background-color: #f1c40f; }
  .tooltiptext.bullet { background-color: #e74c3c; }
</style>

