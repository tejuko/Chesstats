<script>
    export let leaderboardData = null;
    export let username = '';
    export let initialTab = 'live_blitz';

    let activeTab = initialTab;

    const tabs = [
        { key: 'live_rapid', label: 'Rapid' },
        { key: 'live_blitz', label: 'Blitz' },
        { key: 'live_bullet', label: 'Bullet' }
    ];
</script>

<div class="leaderboard">
    <h2>Chess.com Leaderboard</h2>

    {#if !leaderboardData}
        <p class="loading">Loading leaderboard...</p>
    {:else}
        <!-- Tabs -->
        <div class="tabs">
            {#each tabs as tab}
                <button
                    class:active-tab={activeTab === tab.key}
                    on:click={() => activeTab = tab.key}
                >
                    {tab.label}
                </button>
            {/each}
        </div>

        <!-- Leaderboard -->
        <table>
            <thead>
                <tr>
                    <th>Rank</th>
                    <th>Username</th>
                    <th>Score</th>
                </tr>
            </thead>
            <tbody>
                {#each leaderboardData[activeTab].slice(0, 10) as player, index}
                    <tr class:highlight={player.username.toLowerCase() === username.toLowerCase()}>
                        <td>{index + 1}</td>
                        <td>{player.username}</td>
                        <td>{player.score}</td>
                    </tr>
                {/each}

                <!-- If user is outside top 10 -->
                {#if !leaderboardData[activeTab].slice(0, 10).some(p => p.username.toLowerCase() === username.toLowerCase())}
                    {#each leaderboardData[activeTab] as player, idx (player.username)}
                        {#if player.username.toLowerCase() === username.toLowerCase()}
                            <tr class="highlight">
                                <td>{idx + 1}</td>
                                <td>{player.username}</td>
                                <td>{player.score}</td>
                            </tr>
                        {/if}
                    {/each}

                   {#if !leaderboardData[activeTab].some(p => p.username.toLowerCase() === username.toLowerCase())}
                        <tr class="highlight">
                            <td>99+</td>
                            <td>{username}</td>
                            <td>N/A</td>
                        </tr>
                    {/if}
                {/if}
            </tbody>
        </table>
    {/if}
</div>

<style>
    h2 {
        margin: 1rem;
    }

    .leaderboard {
        width: 100%;
        height: fit-content;
        background: linear-gradient(145deg, #ffffff 0%, #f8f8f8 100%);
        padding: 2rem;
        border-radius: 16px;
        box-shadow: 0 6px 15px rgba(0, 0, 0, 0.08);
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .tabs {
        display: flex;
        gap: 1rem;
        justify-content: center;
        margin-bottom: 1rem;
    }

    .tabs button {
        padding: 0.6rem 1.2rem;
        border: none;
        border-radius: 12px;
        background: #f1f1f1;
        cursor: pointer;
        font-weight: 600;
        transition: all 0.2s ease;
    }

    .tabs button.active-tab {
        background: #69923e;
        color: white;
    }

    .tabs button:hover {
        background-color: burlywood;
    }

    table {
        width: 100%;
        border-collapse: collapse;
    }

    th, td {
        padding: 0.8rem;
        text-align: center;
        border-bottom: 1px solid #eee;
    }

    th {
        background-color: #f1f1f1;
    }

    tr:hover {
        background-color: #f1f1f1;
    }

    .highlight {
        background-color: #adcf9b;
        font-weight: 600;
        transition: 0.3s;
    }

    .highlight:hover {
        background-color: #ffd55f;
    }
</style>
