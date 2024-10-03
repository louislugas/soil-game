<script>
	let worlds = [
		{
			w:"Farmland",
			l:1,
		},
		{
			w:"Mountain",
			l:2,
		},
		{
			w:"Forest",
			l:3,
		}
	]
	export let page
	export let world

	import titleImage from "$lib/images/title.png"

	import mountIcon from "$lib/images/mountain_icon.png"
	import forestIcon from "$lib/images/forest_icon.png"
	import farmIcon from "$lib/images/farm_icon.png"

	// function toLevel(e) {
		// page = 2
		// world = e.target.dataset.world
		// lWorld = e.target.dataset.l
		// l = e.target.dataset.level
    //     isMenu = false
	// }

	function startGame(e) {
		// l = e.target.dataset.level
		world = e.target.dataset.world
        page = 2
    }
</script>

<div class="header-container">
	<img src={titleImage} alt="soil defender title header	" >
</div>

<div class="container">
	{#each worlds as w,i}
		<div class="card" on:click={startGame} data-world={w.w}>
			<div class="card-title" data-world={w.w}>
				<h2>{w.w}</h2>
			</div>
			<div class="card-image" data-world={w.w}>
				<img src={w.w == "Forest" ? forestIcon : w.w == "Mountain" ? mountIcon : farmIcon} data-world={w.w}>
			</div>
			<div class="card-button" data-world={w.w}>
				<button data-world={w.w} data-l={w.l} on:click={startGame}>Start</button>
			</div>
		</div>
	{/each}
</div>

<style>
	.header-container {
		/* background-color: indianred; */
		width:70%;
		aspect-ratio: 2;
		margin-bottom: 2rem;
	}
	.container {
		display:flex;
		/* background-color:lightblue; */
		width:100%;
		max-width:500px;
		justify-content:space-around;
		align-items:center;
	}
	.card {
		display:flex;
		flex-direction:column;
		width:30%;
		background-color: #f0dc6a;
		color:#554225;
		box-shadow:2px 2px black;
		cursor: pointer;
		border-radius: 0.3rem;
		border:2px solid black;
	}
	.card-image {
		width:100%;
		aspect-ratio:1;
		/* background-color:lightcoral; */
		justify-content:center;
		align-items:center;
	}
	img {
		width:100%;
		height:100%;
		image-rendering: crisp-edges;
		image-rendering:pixelated;
		image-rendering: -moz-crisp-edges;
	}
	.card-title {
		width:100%;
		margin-top:0.3rem;
		aspect-ratio:3;
		/* background-color:orangered; */
	}
	.card-button {
		width:100%;
		aspect-ratio:3/2;
		/* background-color:lightseagreen; */
		display:flex;
		justify-content:center;
		align-items:center;
	}
	button {
		background-color: transparent;
		border:none;
		font-size:1.2rem;
		/* color:#0e81d1; */
	}
	h2 {
		text-align:center;
		margin:0;
		font-family: 'Jua',sans-serif;
	}
</style>