<script>
    export let page

    import titleImage from "$lib/images/title.png"
</script>

<div class="header-container">
	<img src={titleImage} alt="soil defender title header	" >
</div>

<button on:click={() => {page = 1}}>
Start
</button>

<style>
    .header-container {
		/* background-color: indianred; */
		width:70%;
		aspect-ratio: 2;
		margin-bottom: 2rem;
	}
    img {
		width:100%;
		height:100%;
		image-rendering: crisp-edges;
		image-rendering:pixelated;
		image-rendering: -moz-crisp-edges;
	}
    button {
        background-color: #f0dc6a;
        padding:0.5rem;
        border: 2px solid black;
        box-shadow: 2px 2px black;
        /* color:#0e81d1; */
        aspect-ratio: 2;
        font-size: 1.5rem;
        border-radius: 0.5rem;
    }
</style>