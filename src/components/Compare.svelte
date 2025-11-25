<script>
    import { onMount, afterUpdate } from "svelte"; // Lifecycle hooks
    import * as d3 from "d3"; // D3.js for charting

    export let username = ""; // Logged-in user's username

    // State variables
    let friend = "";          // Friend's username to compare
    let myStats = null;       // Logged-in user's stats
    let myProfile = null;     // Logged-in user's profile
    let friendStats = null;   // Friend's stats
    let friendProfile = null; // Friend's profile

    let loadingMyProfile = false; // Flag for loading user profile
    let loadingFriend = false;    // Flag for loading friend profile
    let error = "";               // Error message

    let chartContainer; // DOM reference for the D3 chart

    const YOU_COLOR = "#69923e";    // Green for user
    const FRIEND_COLOR = "#c0392b"; // Red for friend

    // Format Unix timestamp into readable date
    function formatDate(ts) {
        if (!ts) return "N/A";
        const d = new Date(ts * 1000);
        return d.toLocaleDateString();
    }

    // Determine bar/text color based on comparison
    function getColor(myRating, friendRating, isMy) {
        if (myRating == null || friendRating == null) return "#666"; // Neutral gray if missing
        if (myRating > friendRating) return isMy ? YOU_COLOR : FRIEND_COLOR;
        if (myRating < friendRating) return isMy ? FRIEND_COLOR : YOU_COLOR;
        return "#666"; // Equal ratings
    }

    // Fetch profile and stats from Chess.com API
    async function fetchProfileAndStats(user) {
        const result = { profile: null, stats: null, error: "" };
        const uname = user.trim().toLowerCase();
        if (!uname) {
            result.error = "Username cannot be empty.";
            return result;
        }

        try {
            const pRes = await fetch(`https://api.chess.com/pub/player/${uname}`);
            if (pRes.ok) result.profile = await pRes.json();
            else result.error += `Profile "${uname}" not found. `;

            const sRes = await fetch(`https://api.chess.com/pub/player/${uname}/stats`);
            if (sRes.ok) result.stats = await sRes.json();
            else result.error += "Stats not found.";
        } catch (err) {
            console.error(`Failed to fetch ${uname}:`, err);
            result.error = "Failed to fetch data. Check your connection or try again.";
        }

        return result;
    }

    // Load logged-in user's profile
    async function loadMyProfile() {
        if (!username) return;
        loadingMyProfile = true;
        error = "";
        const { profile, stats, error: err } = await fetchProfileAndStats(username);
        myProfile = profile;
        myStats = stats;
        loadingMyProfile = false;
        if (err) error = `Your profile: ${err}`;
    }

    onMount(loadMyProfile); // Load on component mount

    $: if (username) loadMyProfile(); // Reactive reload when username changes

    // Compare logged-in user vs friend
    async function compare() {
        if (!friend) {
            error = "Enter a friend's username to compare.";
            return;
        }

        error = "";
        loadingFriend = true;
        friendProfile = null;
        friendStats = null;

        const { profile, stats, error: err } = await fetchProfileAndStats(friend);
        friendProfile = profile;
        friendStats = stats;

        if (err) error = err;
        loadingFriend = false;
    }

    function rating(stats, key) {
        return stats?.[key]?.last?.rating ?? null;
    }

    function diffText(my, other) {
        if (my == null || other == null) return "N/A";
        const diff = my - other;
        return (diff > 0 ? "+" : "") + diff;
    }

    function diffClass(my, other) {
        if (my == null || other == null) return "neutral";
        return my > other ? "positive" : my < other ? "negative" : "neutral";
    }

    afterUpdate(() => {
        if (chartContainer && myStats && friendStats) drawChart();
    });

    function drawChart() {
        if (!myStats || !friendStats) return;
        chartContainer.innerHTML = "";

        const data = [
            { mode: "Rapid", my: rating(myStats, "chess_rapid"), friend: rating(friendStats, "chess_rapid") },
            { mode: "Blitz", my: rating(myStats, "chess_blitz"), friend: rating(friendStats, "chess_blitz") },
            { mode: "Bullet", my: rating(myStats, "chess_bullet"), friend: rating(friendStats, "chess_bullet") }
        ].filter(d => d.my !== null || d.friend !== null);

        if (data.length === 0) return;

        const width = 520, height = 260, margin = { top: 20, right: 20, bottom: 40, left: 40 };

        const svg = d3.select(chartContainer)
            .append("svg")
            .attr("width", width)
            .attr("height", height);

        const x0 = d3.scaleBand().domain(data.map(d => d.mode)).range([margin.left, width - margin.right]).padding(0.3);
        const x1 = d3.scaleBand().domain(["my", "friend"]).range([0, x0.bandwidth()]).padding(0.15);
        const y = d3.scaleLinear()
            .domain([0, d3.max(data.flatMap(d => [d.my, d.friend].filter(v => v !== null))) || 1500])
            .nice()
            .range([height - margin.bottom, margin.top]);
        const color = d3.scaleOrdinal().domain(["my", "friend"]).range([YOU_COLOR, FRIEND_COLOR]);

        svg.append("g")
            .selectAll("g")
            .data(data)
            .enter()
            .append("g")
            .attr("transform", d => `translate(${x0(d.mode)},0)`)
            .selectAll("rect")
            .data(d => ["my", "friend"].map(key => ({ key, value: d[key] })))
            .enter()
            .append("rect")
            .attr("x", d => x1(d.key))
            .attr("y", height - margin.bottom)
            .attr("width", x1.bandwidth())
            .attr("height", 0)
            .attr("fill", d => color(d.key))
            .transition()
            .duration(600)
            .attr("y", d => d.value !== null ? y(d.value) : height - margin.bottom)
            .attr("height", d => d.value !== null ? height - margin.bottom - y(d.value) : 0);

        svg.append("g").attr("transform", `translate(0,${height - margin.bottom})`).call(d3.axisBottom(x0));
        svg.append("g").attr("transform", `translate(${margin.left},0)`).call(d3.axisLeft(y));
    }
</script>

<div class="compare-card">
    <h2>Compare Your Stats With a Friend</h2>

    <!-- Loading indicator for user's profile -->
    {#if loadingMyProfile}
        <p class="loading">Loading your profile...</p>
    {/if}

    <!-- Input field for friend's username and Compare button -->
    <div class="input-row">
        <input
            placeholder="friend's username..."
            bind:value={friend} 
            on:keydown={(e) => e.key === 'Enter' && compare()}
        />
        <button on:click={compare} disabled={loadingFriend}>Compare</button>
    </div>

    <!-- Loading indicator for friend's profile -->
    {#if loadingFriend}
        <p class="loading">Loading friend's data...</p>
    {/if}

    <!-- Display any errors -->
    {#if error}
        <p class="error">{error}</p>
    {/if}

    <!-- Display both profiles if loaded -->
    {#if myProfile && friendProfile}
    <div class="profiles">
        <!-- User profile card -->
        <div class="profile-card">
            <img class="avatar" src={myProfile?.avatar ?? '/default-avatar.png'} alt="Your avatar" />
            <div class="profile-info">
                <div class="name">{myProfile?.username ?? 'Unknown'}</div>
                <div class="meta">{myProfile?.title ?? "No title"} • {myProfile?.location ?? ""}</div>
                <div class="small">Joined: {formatDate(myProfile?.joined)} • Last seen: {formatDate(myProfile?.last_online)}</div>
            </div>
        </div>

        <!-- Friend profile card -->
        <div class="profile-card">
            <img class="avatar" src={friendProfile?.avatar ?? '/default-avatar.png'} alt="Friend avatar" />
            <div class="profile-info">
                <div class="name">{friendProfile?.username ?? 'Unknown'}</div>
                <div class="meta">{friendProfile?.title ?? "No title"} • {friendProfile?.location ?? ""}</div>
                <div class="small">Joined: {formatDate(friendProfile?.joined)} • Last seen: {formatDate(friendProfile?.last_online)}</div>
            </div>
        </div>
    </div>

    <!-- Rating cards for each chess mode -->
    <div class="cards">
        {#each ["chess_rapid","chess_blitz","chess_bullet"] as key}
            {@const myRating = rating(myStats, key)}
            {@const friendRating = rating(friendStats, key)}
            <div class="rating-card">
                <div class="mode">
                    {key === "chess_rapid" ? "⏳ Rapid" : key === "chess_blitz" ? "⚡ Blitz" : "🔫 Bullet"}
                </div>
                <div class="values">
                    <!-- User rating -->
                    <div class="player">
                        <div class="label">You</div>
                        <div class="value" style="color: {getColor(myRating, friendRating, true)}">{myRating ?? "Unrated"}</div>
                    </div>
                    <div class="vs">vs</div>
                    <!-- Friend rating -->
                    <div class="player friend">
                        <div class="label">Friend</div>
                        <div class="value" style="color: {getColor(myRating, friendRating, false)}">{friendRating ?? "Unrated"}</div>
                    </div>
                </div>
                <!-- Rating difference badge -->
                <div class="difference {diffClass(myRating, friendRating)}">
                    {#if myRating != null && friendRating != null}
                        {diffText(myRating, friendRating)}
                    {:else}
                        N/A
                    {/if}
                </div>
            </div>
        {/each}
    </div>

    <!-- D3 Bar Chart showing ratings comparison -->
    <div class="chart-card">
        <h3>Rating Comparison</h3>
        <div bind:this={chartContainer} class="chart"></div>
    </div>
{/if}
</div>

<style>
    /* Your original styles remain unchanged */
    h2 {
        text-align: center;
        margin: 1rem;
    }

    .input-row {
        padding-bottom: 1em;
        display: flex;
        gap: 0.8rem;
        justify-content: center;
    }

    input {
        padding: 0.6rem 1rem;
        border-radius: 10px;
        border: 1px solid #dcdfe6;
        width: 60%;
        font-size: 1rem;
        outline: none;
    }

    button {
        padding: 0.55rem 1rem;
        border: none;
        border-radius: 10px;
        background-color: #69923e;
        color: white;
        cursor: pointer;
        font-weight: 600;
    }

    button:hover:not(:disabled) {
        filter: brightness(1.05);
    }

    button:disabled {
        opacity: 0.6;
        cursor: not-allowed;
    }

    .loading, .error {
        text-align: center;
        font-weight: 600;
    }

    .error {
        color: #c0392b;
    }

    .profiles {
        display: flex;
        gap: 1rem;
        justify-content: center;
        padding-bottom: 1rem;
    }

    .profile-card {
        display: flex;
        gap: 0.8rem;
        align-items: center;
        background: #fff;
        padding: 0.8rem;
        border-radius: 12px;
        width: 48%;
        box-shadow: 0 4px 12px rgba(0,0,0,0.04);
    }

    .avatar {
        width: 64px;
        height: 64px;
        border-radius: 12px;
        object-fit: cover;
    }

    .profile-info .name { font-weight: 700; }
    .profile-info .meta { color: #666; font-size: 0.9rem; }
    .profile-info .small { font-size: 0.78rem; color: #98a0aa; }

    .cards {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        flex-wrap: wrap;
    }

    .rating-card {
        background: #fff;
        border-radius: 12px;
        padding: 1rem;
        width: fill;
        min-width: 180px;
        box-shadow: 0 6px 18px rgba(38, 57, 84, 0.04);
        position: relative;
        display: flex;
        flex-direction: column;
        gap: 0.6rem;
    }

    .mode { font-weight: 800; }
    .values { display: flex; justify-content: space-between; align-items: center; }
    .player { display:flex; flex-direction: column; align-items: center; }
    .player .label { font-size: 1em; color: #7b8794;  }
    .player .value { font-weight: 800; font-size: 1.4rem; color: #69923e; }
    .player.friend .value { color: #c0392b; }
    .vs { color: #9aa3b2; font-weight: 700; font-size: 3em; }
    .difference {
        width: fit-content;
        padding: 0.35rem 0.6rem;
        border-radius: 999px;
        font-weight: 700;
        font-size: 0.95rem;
    }
    .difference.positive { background: #d4efd7; color: #69923e; }
    .difference.negative { background: #f9d6d6; color: #c0392b; }
    .difference.neutral { background: #f1f5f9; color: #6b7280; }

    .chart-card {
        display: flex;
        justify-content: center;
        flex-direction: column;
        align-items: center;
        margin-top: 1rem;
        background: #fff;
        border-radius: 12px;
        padding: 0.8rem;
        box-shadow: 0 6px 18px rgba(38, 57, 84, 0.04);
    }

    .chart { width: 100%; height: 260px; }

    @media (max-width: 900px) {
        .profiles { flex-direction: column; }
        .profile-card { width: 100%; }
        .cards { flex-direction: column; }
        .rating-card { width: 100%; }
    }
</style>