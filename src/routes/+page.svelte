<script>
    import Navbar from '../components/Navbar.svelte';
    import { goto } from '$app/navigation';

    let username = '';
    let error = '';

    async function login() {
        error = '';
        if (!username) {
            error = 'Username is required';
            return;
        }

        try {
            const res = await fetch(`https://api.chess.com/pub/player/${username}`);
            if (res.ok) {
                localStorage.setItem('username', username);
                goto('/dashboard');
            } else {
                error = 'Username not found on Chess.com';
            }
        } catch (err) {
            error = 'Something went wrong';
            console.error(err);
        }
    }

    function handleKeyDown(event) {
        if (event.key === 'Enter') {
            login();
        }
    }

    function loginAsTopPlayer(playerName) {
    username = playerName;
    login();
}
</script>

<svelte:head>
    <title>Login - Chesstats.com</title>
    <meta name="description" content="Login to Chesstats.com to view your chess statistics." />
</svelte:head>

<Navbar />

<div class="page-container">
    <div class="login-card">
        <img src="/logo1.png" alt="Logo van chesstats.com" />
        <h2>Chess.com Login</h2>
        {#if error}
            <div class="error">{error}</div>
        {/if}
        <input
            type="text"
            placeholder="Enter Chess.com username"
            bind:value={username}
            on:keydown={handleKeyDown}
        />
        <button on:click={login}><strong>Login</strong></button>
        <h3>Or pick one of our top players!</h3>
        <div class="top-players">
            <button on:click={() => loginAsTopPlayer('Hikaru')}><strong>Hikaru</strong></button>
            <button on:click={() => loginAsTopPlayer('MagnusCarlsen')}><strong>MagnusCarlsen</strong></button>
            <button on:click={() => loginAsTopPlayer('FabianoCaruana')}><strong>FabianoCaruana</strong></button>
        </div>
    </div>
</div>

<style>
    .top-players {
        display: flex;
        gap: 1rem;
    }

    .page-container {
        display: flex;
        justify-content: center;
        align-items: center;
        margin-top: 5em;
        background-color: burlywood;
    }

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

    .login-card img {
        width: 100px;
    }

    input {
        width: 100%;
        padding: 0.5rem;
        margin-bottom: 1rem;
        border-radius: 6px;
        border: 1px solid #ccc;
        font-size: 1rem;
    }

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

    button:hover {
        background-color: #4e7837;
    }

    .error {
        color: red;
        margin-bottom: 1rem;
    }
</style>