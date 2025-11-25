<script>
    // Import reusable components for the dashboard page
    import Navbar from '../../components/Navbar.svelte';
    import Dashboard from '../../components/Dashboard.svelte';
    import ChessChart from '../../components/ChessChart.svelte';
    import Leaderboard from '../../components/Leaderboard.svelte';
    import Compare from '../../components/Compare.svelte';

    // Svelte lifecycle function
    import { onMount } from 'svelte';

    // Navigation utility from SvelteKit
    import { goto } from '$app/navigation';

    // State variables
    let username = '';          // Stores the current user's Chess.com username
    let playerData = null;      // Stores player's chess stats fetched from API
    let profileData = null;     // Stores player's profile info fetched from API
    let leaderboardData = null; // Stores leaderboard data fetched from API

    let loading = true;         // Loading state for player stats
    let lbLoading = true;       // Loading state for leaderboard

    let error = '';             // Error message for player stats
    let lbError = '';           // Error message for leaderboard

    // Tabs for leaderboard display (Rapid, Blitz, Bullet)
    const tabs = [
        { key: 'live_rapid', label: 'Rapid' },
        { key: 'live_blitz', label: 'Blitz' },
        { key: 'live_bullet', label: 'Bullet' }
    ];

    // Runs when the component mounts
    onMount(async () => {
        // Retrieve username from localStorage
        username = localStorage.getItem('username');
        
        // If username doesn't exist, redirect to login page
        if (!username) return goto('/');

        // Prepare API endpoints
        const endpoints = [
            fetch(`https://api.chess.com/pub/player/${username}/stats`), // Player stats
            fetch(`https://api.chess.com/pub/player/${username}`),       // Player profile
            fetch('https://api.chess.com/pub/leaderboards')             // Leaderboards
        ];

        try {
            // Perform all requests in parallel
            const [statsRes, profileRes, lbRes] = await Promise.all(endpoints);

            // Handle player stats response
            if (statsRes.ok) {
                playerData = await statsRes.json();
            } else {
                error = `Player data not found (Status: ${statsRes.status})`;
            }

            // Handle profile response
            if (profileRes.ok) {
                profileData = await profileRes.json();
            } else {
                console.warn('Profile request failed:', profileRes.status);
            }

            // Handle leaderboard response
            if (lbRes.ok) {
                leaderboardData = await lbRes.json();
            } else {
                lbError = `Failed to load leaderboard (Status: ${lbRes.status})`;
            }

        } catch (err) {
            // Catch network errors or unexpected issues
            console.error('Fetch error', err);

            // Display errors only if data wasn't loaded
            error = playerData ? '' : 'Failed to load player data';
            lbError = leaderboardData ? '' : 'Failed to load leaderboard';
        } finally {
            // Stop loading indicators regardless of success or failure
            loading = false;
            lbLoading = false;
        }
    });

    // Function to logout user
    function logout() {
        localStorage.removeItem('username'); // Clear stored username
        goto('/'); // Redirect to login page
    }
</script>

<!-- Page head for SEO -->
<svelte:head>
    <title>Dashboard - Chesstats.com</title>
    <meta name="description" content="View your chess statistics on Chesstats.com." />
</svelte:head>

<!-- Navbar component -->
<Navbar />

<!-- Welcome section -->
<div class="welcome-wrapper">
    <div class="welcome-header">
        {#if profileData?.avatar}
            <!-- Show user avatar if available -->
            <img src={profileData.avatar} alt={username} class="avatar" />
        {/if}
        <h1>Welcome, {username}!</h1>
    </div>
</div>

<!-- Main dashboard section -->
<div class="page-container">
    <!-- Dashboard card showing stats -->
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
        <!-- Chess chart card -->
        <div class="chart-card">
            <ChessChart {username} year={2025} />
        </div>
    {/if}
</div>

<!-- Leaderboard and Compare section -->
<div class="page-container">
    <!-- Leaderboard component -->
    <Leaderboard 
        {leaderboardData} 
        {username}
        initialTab="live_blitz"
    />

    <!-- Compare component for comparing players -->
    <div class="compare">
        <Compare {username} />
    </div>
</div>

<style>
    /* Welcome section styling */
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

    /* Avatar image styling */
    .avatar {
        background-color: white;
        width: 60px;
        height: 60px;
        border-radius: 50%;
        object-fit: cover;
        box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }

    /* Compare component container */
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

    /* Page container for layout */
    .page-container {
        display: flex;
        justify-content: center;
        align-items: stretch;
        gap: 2rem;
        padding: 0em 1em 2em 1em;
        max-width: 1400px;
        margin: 0 auto;
    }

    /* Dashboard and chart card styling */
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

    /* Hover effect for cards */
    .dashboard-card:hover,
    .chart-card:hover {
        transform: translateY(-4px);
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.12);
    }

    /* Fade-in animation */
    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(10px); }
        to { opacity: 1; transform: translateY(0); }
    }

    /* Chart card specific styling */
    .chart-card {
        background: linear-gradient(145deg, #fdfdfd 0%, #f4f4f4 100%);
        position: relative;
        overflow: hidden;
    }

    /* Shimmer effect on chart card */
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

    /* Chart SVG styling */
    .chart-card > :global(svg) {
        height: 320px;
        width: 100%;
        border-radius: 12px;
        position: relative;
        background-color: white;
        box-shadow: 0 3px 10px rgba(0,0,0,0.05);
    }
</style>