<script>
    export let page, world, isMenu
    import instructionImg1 from "$lib/images/instruction-1.png"
    import instructionImg2 from "$lib/images/instruction-2.png"
</script>

<section>
    <div class="top">
    <div class="image">
        <img src={instructionImg1} alt="attack instruction">
    </div>
    <div class="image">
        <img src={instructionImg2} alt="defense instruction">
    </div>
    </div>
    <div class="bottom">
        <p><strong>Defend soil health!</strong><br><br>Use the <em><strong>plant</strong></em> to launch projectiles at the enemy, or the <em><strong>rock</strong></em> to block them from advancing. Click anywhere on the grid to place your <em><strong>plant</strong></em> or <em><strong>rock</strong></em>.<br><br>May the dirt be with you!</p>
    </div>
    <button on:click={() => {isMenu = false}}>
    Start
    </button>
</section>

<style>
    section {
		display:flex;
		width:100%;
        flex-direction: column;
		max-width:500px;
		justify-content:space-around;
		align-items:center;
	}
    .top {
        width:90%;
        display: flex;
        justify-content: space-around;
        align-items: center;
    }
    .bottom {
        width:80%;
        box-shadow:2px 2px black;
		border-radius: 0.3rem;
		border:2px solid black;
        margin-top: 1rem;
        padding:0.5rem;
        font-family:"Nunito", sans-serif;
        background-color: #f9eeb0;
    }
    .image {
        width:45%;
        aspect-ratio:2/3;
        /* background-color: red; */
        overflow: hidden;
        box-shadow:2px 2px black;
		border-radius: 0.3rem;
		border:2px solid black;
    }
    img {
        width:100%;
        height:100%;
        object-fit: contain;
    }
    button {
        margin-top: 1rem;
        background-color: #f0dc6a;
        padding:0.5rem;
        border: 2px solid black;
        box-shadow: 2px 2px black;
        /* color:#0e81d1; */
        aspect-ratio: 2;
        font-size: 1.5rem;
        border-radius: 0.5rem;
    }
    p {
        margin:0;
    }
</style>