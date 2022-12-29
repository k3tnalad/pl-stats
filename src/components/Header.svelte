<script>
    let TITLE = 'Premier League Lobby';
    import '/src/app.css';
</script>

<header>
    <h1>{TITLE.toUpperCase()}</h1>
</header>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Abel&display=swap');
    header {
        width: 100%;
        padding: 2em;
        font-family: 'Abel', sans-serif;
        background-color: #FBFBFF;
        text-align: left;
    }

    @media screen and (max-width: 640px) {
        header {
            text-align: center;
        }
    }
</style>