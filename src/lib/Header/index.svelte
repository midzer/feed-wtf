<script>
    import '../../vendor/css/choices.css';
	import './index.css';

	import { onMount } from "svelte";
	import { browser } from '$app/env';
	import { page } from '$app/stores';
	//import { goto, invalidate, prefetch, prefetchRoutes } from '$app/navigation';
	import slugo from 'slugo';

	let city;
	let user;
  
	function handleUserClick (event) {
    	//goto(`/@${user}`);
		// TODO: this fix is a hack, see https://github.com/sveltejs/kit/issues/552
		window.location = `/@${slugo(user)}`;
	}

	function handleCityClick (event) {
    	//goto(`/${city}`);
		// TODO: this fix is a hack, see https://github.com/sveltejs/kit/issues/552
		window.location = `/${slugo(city)}`;
	}

	async function fetchCities () {
		return fetch('/data/cities.json')
		.then(function(response) {
			return response.json();
		})
		.then(function(data) {
			return data.map(function(item) {
				return {
					value: item,
					label: item
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
				placeholderValue: 'City',
				searchFields: ['value'],
				searchPlaceholderValue: 'Searching...',
				loadingText: 'Loading...',
				noResultsText: 'No results found',
				itemSelectText: '+',
				noChoicesText: 'with at least 3 letters',
				renderChoiceLimit : 0,
				searchResultLimit: 100,
				shouldSort: false,
				position: 'bottom'
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
						return fetchCities();
					})
					.then(function() {
						// We have to re-set focus on input again
						instance.input.element.focus();
					});
				}, { once: true });
				instance.passedElement.element.addEventListener('change', function(e) {
					city = e.detail.value;
				});
			});
		});
	};
</script>

<header>
	<h1><a href="/" aria-label="Navigate to home page">📰</a> feedi.net ALPHA - {$page.params.city || $page.params.user || 'Local news that matter'}</h1>
	<section>
		<label for="user">Choose</label>
    	<input type="text" id="user" name="user" placeholder="Nickname" bind:value={user}>
		<button id="userButton" disabled={user ? false : true} on:click={handleUserClick}>Go!</button>
		<label for="cities">or</label>
    	<select data-type="select-one" class="form-control" name="cities" id="cities"></select>
    	<button id="cityButton" disabled={city ? false : true} on:click={handleCityClick}>Go!</button>
  	</section>
</header>
