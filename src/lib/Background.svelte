<section>
    <div>
        <slot />
    </div>
</section>

<style>
    :root {
		--width:200px;
		--m:2;
	}
	section {
        width:100%;
        height: 100%;
		/* background-color:cornflowerblue; */
		display:flex;
		justify-content:center;
		align-items:center;
		flex-direction:column;
		image-rendering: crisp-edges;
		background-image:url('$lib/images/background.png');
		/* Must be a square */
		background-size:var(--width) var(--width);
		/* Must be half of one side of the square */
		background-position:0px 0px;
		animation: bgMove 8000ms linear infinite;
	}

	@keyframes bgMove {
		0% {
			background-position:
				0 0
		}
		100% {
			background-position:
				calc(var(--width) * -1) calc(var(--width) * -1)
		}
	}
    div {
        width:100%;
        max-width: 500px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
</style>