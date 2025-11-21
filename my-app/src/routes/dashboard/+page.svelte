<script>
    import Header from '../../components/Header.svelte';
    import Dashboard from '../../components/Dashboard.svelte';
    import ChessChart from '../../components/ChessChart.svelte';
    import Leaderboard from '../../components/Leaderboard.svelte';
    import Compare from '../../components/Compare.svelte';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    let username = '';
    let playerData = null;
    let profileData = null;
    let leaderboardData = null;

    let loading = true;
    let lbLoading = true;

    let error = '';
    let lbError = '';

    const tabs = [
        { key: 'live_rapid', label: 'Rapid' },
        { key: 'live_blitz', label: 'Blitz' },
        { key: 'live_bullet', label: 'Bullet' }
    ];

    onMount(async () => {
        username = localStorage.getItem('username');
        if (!username) return goto('/');

        const endpoints = [
            fetch(`https://api.chess.com/pub/player/${username}/stats`),
            fetch(`https://api.chess.com/pub/player/${username}`),
            fetch('https://api.chess.com/pub/leaderboards')
        ];

        try {
            const [statsRes, profileRes, lbRes] = await Promise.all(endpoints);

            // Player Stats
            if (statsRes.ok) {
                playerData = await statsRes.json();
            } else {
                error = `Player data not found (Status: ${statsRes.status})`;
            }

            // Profile
            if (profileRes.ok) {
                profileData = await profileRes.json();
            } else {
                console.warn('Profile request failed:', profileRes.status);
            }

            // Leaderboards
            if (lbRes.ok) {
                leaderboardData = await lbRes.json();
            } else {
                lbError = `Failed to load leaderboard (Status: ${lbRes.status})`;
            }

        } catch (err) {
            console.error('Fetch error', err);
            error = playerData ? '' : 'Failed to load player data';
            lbError = leaderboardData ? '' : 'Failed to load leaderboard';
        } finally {
            loading = false;
            lbLoading = false;
        }
    });

    function logout() {
        localStorage.removeItem('username');
        goto('/');
    }
</script>

<svelte:head>
    <title>Dashboard - Chesstats.com</title>
    <meta name="description" content="View your chess statistics on Chesstats.com." />
</svelte:head>

<Header />

<div class="welcome-wrapper">
    <div class="welcome-header">
        {#if profileData?.avatar}
            <img src={profileData.avatar} alt={username} class="avatar" />
        {/if}
        <h1>Welcome, {username}!</h1>
    </div>
</div>

<div class="page-container">
    <div class="dashboard-card">
        <Dashboard 
            {username}
            {playerData} 
            {profileData}
            {loading}
            {error}
        />
    </div>

    {#if username}
        <div class="chart-card">
            <ChessChart {username} year={2025} />
        </div>
    {/if}
</div>

<div class="page-container">
    <Leaderboard 
        {leaderboardData} 
        {username}
        initialTab="live_blitz"
    />

    <div class="compare">
        <Compare {username} />
    </div>
</div>

<style>
    .welcome-wrapper {
        display: flex;
        justify-content: center;
        align-items: center;
    }   

    .welcome-header {
        background: linear-gradient(145deg, #ffffff 0%, #f8f8f8 100%);
        border-radius: 16px;
        box-shadow: 0 6px 15px rgba(0, 0, 0, 0.08);
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 1rem;
        width: fit-content;
        margin: 1em;
        padding: 0em 1em;
    }

    .avatar {
        background-color: white;
        width: 60px;
        height: 60px;
        border-radius: 50%;
        object-fit: cover;
        box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }

    .compare {
        width: 100%;
        background: linear-gradient(145deg, #ffffff 0%, #f8f8f8 100%);
        padding: 2rem;
        border-radius: 16px;
        box-shadow: 0 6px 15px rgba(0, 0, 0, 0.08);
        display: flex;
        flex-direction: column;
        align-items: center;
        animation: fadeIn 0.6s ease forwards;
    }

    .page-container {
        display: flex;
        justify-content: center;
        align-items: stretch;
        gap: 2rem;
        padding: 0em 1em 2em 1em;
        max-width: 1400px;
        margin: 0 auto;
    }

    .dashboard-card,
    .chart-card {
        width: 100%;
        background: linear-gradient(145deg, #ffffff 0%, #f8f8f8 100%);
        padding: 2rem;
        border-radius: 16px;
        box-shadow: 0 6px 15px rgba(0, 0, 0, 0.08);
        display: flex;
        flex-direction: column;
        text-align: center;
        transition: transform 0.25s ease, box-shadow 0.25s ease;
        animation: fadeIn 0.6s ease forwards;
    }

    .dashboard-card:hover,
    .chart-card:hover {
        transform: translateY(-4px);
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.12);
    }

    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(10px); }
        to { opacity: 1; transform: translateY(0); }
    }

    .chart-card {
        background: linear-gradient(145deg, #fdfdfd 0%, #f4f4f4 100%);
        position: relative;
        overflow: hidden;
    }

    .chart-card::before {
        content: '';
        position: absolute;
        top: 0;
        left: -150%;
        width: 50%;
        height: 100%;
        background: linear-gradient(120deg, rgba(255,255,255,0.4), rgba(255,255,255,0));
        transform: skewX(-25deg);
        animation: shimmer 4s infinite;
    }

    .chart-card > :global(svg) {
        height: 320px;
        width: 100%;
        border-radius: 12px;
        position: relative;
        background-color: white;
        box-shadow: 0 3px 10px rgba(0,0,0,0.05);
    }
</style>

