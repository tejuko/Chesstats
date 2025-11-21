<script>
    import { goto } from '$app/navigation';
    import { onMount } from 'svelte';

    let username = '';

    onMount(() => {
        username = localStorage.getItem('username');
    });

    function logout() {
        localStorage.removeItem('username');
        username = '';
        goto('/');
    }

    function goHome() {
        goto('/');
    }

</script>

<style>
    nav {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0.5rem 2rem;
        background: #69923e;
        color: white;
        box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }

    /* Logo section */
    .logo {
        display: flex;
        align-items: center;
        cursor: pointer; /* shows hand cursor */
        background: none;
        border: none;
        padding: 0 0.5em 0 0;
        font: inherit;
    }

    .logo img {
        width: 50px;
        margin-right: 0.5rem;
        transition: transform 0.2s ease;
    }

    .logo img:hover {
        transform: scale(1.1);
    }

    /* Navigation links */
    .nav-links {
        display: flex;
        align-items: center;
        gap: 1.5rem;
    }

    .nav-links a {
        color: white;
        text-decoration: none;
        font-weight: bold;
        font-size: 1.2em;
        position: relative;
    }

    /* Hover underline effect */
    .nav-links a::after {
        content: '';
        display: block;
        height: 2px;
        background: white;
        width: 0;
        transition: width 0.3s;
        position: absolute;
        bottom: -4px;
        left: 0;
    }

    .nav-links a:hover::after {
        width: 100%;
    }

    a:focus, button:focus {
        outline: none;
    }

    button {
        padding: 0.5em 1em;
        background-color: #4e7837;
        color: white;
        border: none;
        border-radius: 0.5em;
        cursor: pointer;
        font-weight: bold;
        font-size: 1.2em;
        transition: background 0.2s ease, transform 0.2s ease;
    }

    button:hover {
        background-color: #4e7837;
        transform: scale(1.05);
    }

    span {
        font-size: 1.2em;
        font-weight: bold;
    }

</style>

<nav>
    <button type="button" class="logo" on:click={goHome} aria-label="Go to homepage">
        <img src="/logo1.png" alt="logo van chesstats.com" />
        <span>Chesstats.com</span>
    </button>

    <!-- Navigation Links -->
    <div class="nav-links">
        <a href="https://www.chess.com/play">♔ Play a game!</a>
        {#if username}
            <button on:click={logout}>Logout</button>
        {/if}
    </div>
</nav>
