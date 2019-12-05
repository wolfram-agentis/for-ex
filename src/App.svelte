<script>
	let amount;
	let baseCurrency;
	let targetCurrency;
	let conversionRatio;
	$: convertedAmount = conversionRatio ? conversionRatio * amount : '...awaiting input';

	let currencyList = getCurrencyList();

	async function getCurrencyList() {
		const response = await fetch('https://api.exchangeratesapi.io/latest');

		return await response.json().then(data => {
			return Object.keys(data.rates).sort((a, b) => a > b ? 1 : -1);
		});
	}

	async function convert(e) {
		const response = await fetch(`https://api.exchangeratesapi.io/latest?symbols=${targetCurrency}&base=${baseCurrency}`);
		conversionRatio = await response.json().then(data => Number(data.rates[targetCurrency]));
	}

	function resetConversionRatio() {
		conversionRatio = amount = null;
	}
</script>

<style>
	pre {
		display: inline-block;
	}
</style>

<main>
	{#await currencyList}
		<h3>loading...</h3>
	{:then currencies}
		<form on:submit|preventDefault={convert}>
			<label>Amount <input type="number" bind:value={amount} min=0></label>
			<label>Base Currency
				<select bind:value={baseCurrency} on:change={resetConversionRatio}>
					{#each currencies as currency}
						<option value={currency}>{currency}</option>
					{/each}
				</select>
			</label>
			<label>Target Currency
				<select bind:value={targetCurrency} on:change={resetConversionRatio}>
					{#each currencies as currency}
						<option value={currency}>{currency}</option>
					{/each}
				</select>
			</label>
			<input type="submit">
		</form>
		<h3>Converted Amount: <pre>{convertedAmount}</pre></h3>
	{/await}
</main>
