<script>
	import './index.css';

	import { onMount } from "svelte";
	import { browser } from '$app/env';
	import { page } from '$app/stores';
	import { goto/*, invalidate, prefetch, prefetchRoutes*/ } from '$app/navigation';
	import slugo from 'slugo';

	import '../../vendor/css/choices.css';
  
	let region;
	let city;
	let user;

	function handleChange (event) {
		user = event.target.value;
		document.getElementById('userButton').disabled = user ? false : true;
	}
  
	function handleUserClick (event) {
    	goto(`/@${slugo(user)}`);
		// TODO: this fix is a hack, see https://github.com/sveltejs/kit/issues/552
		//window.location = `/@${slugo(user)}`;
	}

	function handleCityClick (event) {
    	goto(`/${slugo(region)}/${slugo(city)}`);
		// TODO: this fix is a hack, see https://github.com/sveltejs/kit/issues/552
		//window.location = `/${slugo(region)}/${slugo(city)}`;
	}
  
	async function fetchRegions() {
		const response = await fetch('/data/regions.json');
		return await response.json();
	}

	async function fetchRegion (region) {
		const regionSlug = slugo(region);
		return fetch('/data/' + regionSlug + '.json')
		.then(function(response) {
			return response.json();
		})
		.then(function(data) {
			return data.map(function(item) {
				return {
					value: item,
					label: item + ', ' + region,
					customProperties: {
						region: regionSlug,
					}
				};
			});
		})
		.catch(function(error) {
			console.error(error);
		});
	}
	
	if (browser) {
    	onMount(async () => {
    		const module = await import('choices.js');
			const Choices = module.default;

			const cities = document.getElementById('cities');
			const citiesChoices = new Choices(cities, {
				/*placeholderValue: LANG.choosePlace,
				searchFields: ['value'],
				searchPlaceholderValue: LANG.search,
				loadingText: LANG.loading,
				noResultsText: LANG.noResultsFound,
				itemSelectText: '+',
				noChoicesText: LANG.withAtLeastThreeLetters,
				renderChoiceLimit : 0,
				searchResultLimit: 100,
				shouldSort: false,
				position: 'bottom'*/
			})
			.setChoices(function() {
				return new Promise(function (resolve) {
					resolve();
				});
			})
			.then(function(instance) {
				instance.containerOuter.element.addEventListener('click', function() {
					// Load data after user input
					instance.setChoices(async function() {
						return fetchRegions()
						.then(async function(regions) {
							const promises = regions.map(function(region) {
								return fetchRegion(region);
							});
							return Promise.all(promises)
							.then(function(result) {
								let citiesArray = [];
								result.forEach(function(array) {
									if (array) {
										citiesArray = citiesArray.concat(array);
									}
								});
								return citiesArray;
							})
							.catch(function(error) {
								console.error(error);
							})
						})
					})
					.then(function() {
						// We have to re-set focus on input again
						instance.input.element.focus();
					});
				}, { once: true });
				instance.passedElement.element.addEventListener('change', function(e) {
					region = instance.getValue().customProperties.region;
					city = e.detail.value;
				});
			});
		});
	};
</script>

<header>
	<h1><a href="/" aria-label="Zur Startseite navigieren">📰</a> feed.wtf ALPHA - {$page.params.city || $page.params.user || 'Lokale Nachrichten, die zählen'}</h1>
	<section>
		<label for="user">Wähle</label>
    	<input type="text" id="user" name="user" placeholder="Nickname" bind:value={user}>
		<button id="userButton" disabled={user ? false : true} on:click={handleUserClick}>Los!</button>
		<label for="cities">oder Ort</label>
    	<select data-type="select-one" class="form-control" name="cities" id="cities"></select>
    	<button id="cityButton" disabled={city ? false : true} on:click={handleCityClick}>Los!</button>
  	</section>
</header>
