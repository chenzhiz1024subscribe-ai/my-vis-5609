<script>
	let maxClicks = 5;
	let usedClicks = 0;

	// 使用一次点击
	function increase() {
		if (usedClicks < maxClicks) {
			usedClicks += 1;
		} else {
			alert("No more clicks remaining!");
		}
	}

	// 还回一次点击
	function decrease() {
		if (usedClicks > 0) {
			usedClicks -= 1;
		}
	}

	// 计算剩余点击次数
	$: remaining = maxClicks - usedClicks;
</script>

<h1>Click Counter</h1>

<p>Remaining Clicks: {remaining}</p>

<button on:click={increase}>
	Increase
</button>

<button on:click={decrease}>
	Decrease
</button>

<br /><br />

<label>
	Set Max Clicks:
	<input type="number" bind:value={maxClicks} min="1" />
</label>

<style>
	button {
		background-color: #44aa66;
		font-size: xx-large;
		margin-right: 10px;
		padding: 10px;
		border: none;
		cursor: pointer;
	}

	button:hover {
		opacity: 0.85;
	}
</style>
