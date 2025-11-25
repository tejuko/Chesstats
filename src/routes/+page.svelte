<script>
    // Import the Navbar component from the components folder
    import Navbar from '../components/Navbar.svelte';

    // Import the navigation function from SvelteKit to programmatically navigate pages
    import { goto } from '$app/navigation';

    // State variables
    let username = ''; // stores the user's input username
    let error = ''; // stores any error message to display

    // Function to handle login logic
    async function login() {
        error = ''; // Reset any previous errors

        // Simple validation: check if the username is empty
        if (!username) {
            error = 'Username is required';
            return;
        }

        try {
            // Fetch user data from the Chess.com public API
            const res = await fetch(`https://api.chess.com/pub/player/${username}`);

            if (res.ok) {
                // If user exists, save username in localStorage and navigate to dashboard
                localStorage.setItem('username', username);
                goto('/dashboard');
            } else {
                // If user not found
                error = 'Username not found on Chess.com';
            }
        } catch (err) {
            // Catch network or other errors
            error = 'Something went wrong';
            console.error(err);
        }
    }

    // Function to allow pressing "Enter" to login
    function handleKeyDown(event) {
        if (event.key === 'Enter') {
            login();
        }
    }

    // Function to login quickly as a top player
    function loginAsTopPlayer(playerName) {
        username = playerName; // Set the username to the chosen top player
        login(); // Call login function
    }
</script>

<!-- Head section for SEO -->
<svelte:head>
    <title>Login - Chesstats.com</title>
    <meta name="description" content="Login to Chesstats.com to view your chess statistics." />
</svelte:head>

<!-- Navbar component -->
<Navbar />

<!-- Main login card -->
<div class="page-container">
    <div class="login-card">
        <!-- Website logo -->
        <img src="/logo1.png" alt="Logo van chesstats.com" />
        <h2>Chess.com Login</h2>

        <!-- Display error message if exists -->
        {#if error}
            <div class="error">{error}</div>
        {/if}

        <!-- Input field for username -->
        <input
            type="text"
            placeholder="Enter Chess.com username"
            bind:value={username}
            on:keydown={handleKeyDown}
        />

        <!-- Login button -->
        <button on:click={login}><strong>Login</strong></button>

        <h3>Or pick one of our top players!</h3>
        <div class="top-players">
            <!-- Buttons for quick login as top players -->
            <button on:click={() => loginAsTopPlayer('Hikaru')}><strong>Hikaru</strong></button>
            <button on:click={() => loginAsTopPlayer('MagnusCarlsen')}><strong>MagnusCarlsen</strong></button>
            <button on:click={() => loginAsTopPlayer('FabianoCaruana')}><strong>FabianoCaruana</strong></button>
        </div>
    </div>
</div>

<style>
    /* Top players button container */
    .top-players {
        display: flex;
        gap: 1rem;
    }

    /* Center the login page content */
    .page-container {
        display: flex;
        justify-content: center;
        align-items: center;
        margin-top: 5em;
        background-color: burlywood;
    }

    /* Styling the login card */
    .login-card {
        display: flex;
        flex-direction: column;
        align-items: center;
        background-color: white;
        padding: 2rem;
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        text-align: center;
        width: auto;
    }

    /* Logo image styling */
    .login-card img {
        width: 100px;
    }

    /* Input field styling */
    input {
        width: 100%;
        padding: 0.5rem;
        margin-bottom: 1rem;
        border-radius: 6px;
        border: 1px solid #ccc;
        font-size: 1rem;
    }

    /* Button styling */
    button {
        width: 100%;
        padding: 0.7rem;
        background-color: #69923e;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 1rem;
    }

    /* Button hover effect */
    button:hover {
        background-color: #4e7837;
    }

    /* Error message styling */
    .error {
        color: red;
        margin-bottom: 1rem;
    }
</style>