<script>
	import FeatureSelect from './FeatureSelect.svelte';
	// export let dataset;
	export let Groupby;

	const dropdown_options = ["None", "Metric", "Category", "Conference"]
</script>

<div class="dropdown-container">
	<div class="dropdown-box">
		<FeatureSelect bind:value={Groupby} options={dropdown_options} label={'Group By'} />
	</div>
</div>

<style>
	/* place the dropdown menus next to each other with 16px of space in between */
	.dropdown-container {
		text-align: center;
	 }

	.dropdown-box {
		border: 5px solid #ccc; 
		padding: 10px;
		display: inline-block; 
		border-radius: 5px;
	}
</style>
