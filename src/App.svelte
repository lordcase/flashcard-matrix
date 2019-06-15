<script>
	import Solvable from './Solvable.svelte'
	import {correct_count} from './stores.js'
	import Cookies from 'js-cookie'
	import { onMount } from 'svelte';
	
	let search_term = ''
	let base_verb = 'base verb...'
	let amount = 7
	let solvable_count = 0
	let searched = false
	let active = false
	let timers
	$correct_count = {0:false}
	const APIURL = 'http://spanish.webfactoo.com/verbos_api.php'
	const moods = ['Imperativo Afirmativo', 'Imperativo Negativo', 'Indicativo', 'Subjuntivo']
	const tenses = ['Condicional', 'Condicional perfecto', 'Futuro', 'Futuro perfecto', 'Imperfecto', 'Pluscuamperfecto', 'Presente', 'Presente perfecto', 'Pretérito', 'Pretérito Anterior']
	const forms = ['1s', '2s', '3s', '1p', '2p', '3p', 'gerund', 'past participle']
	let collection = []
	let solvables = []
	let verbz = []
	let moodz = ['Indicativo']
	let tensez = ['Pretérito', 'Imperfecto', 'Presente']
	let formz = ['1s', '2s','3s','1p', '2p','3p', 'gerund', 'past participle']
	$: if (Object.values($correct_count).every(x=>x===true)) {
		console.log("gronk")
		if (typeof timers === "Object") timers.clearTimeout()
		active = true
		timers = setTimeout(()=>{active=false;console.log("fall")}, 3000)
	}

	onMount(()=>{
		let user_prefs = JSON.parse(Cookies.get('spanish_settings'))
		moodz = user_prefs.moodz.length===0 ? moodz : user_prefs.moodz
		tensez = user_prefs.tensez.length===0 ? tensez : user_prefs.tensez
		formz = user_prefs.formz.length===0 ? formz : user_prefs.formz
		amount = user_prefs.amount ? user_prefs.amount : amount
	})	
	
	function handleSubmit (e) {
		if (e.key === "Enter" && search_term.length > 0 && !e.target.dataset.stop) {
			solvables = []
			e.stopPropagation()
			search(search_term)
		}
	}
	const savePrefs = () => {
		Cookies.set('spanish_settings',JSON.stringify({moodz,tensez,formz,amount}))
	}

	async function search(verb) {
		solvables = []
		searched = true
		base_verb = verb
		const myHeaders = new Headers();
		const myInit = {
			method: `GET`,
			headers: myHeaders,
		}
		fetch(APIURL+`?verb=${verb}`, myInit)
		.then(res => res.json())
		.then(result => {
				console.log("verbs",result)
				collect(result)
		})
		.then(()=> {assemble()})
		.catch(
			(error) => {alert(error.message)}
		)
	}
	async function collect(verbs) {
		collection = []
		let verb
		for (let v in verbs) {
			verb = verbs[v]
			let exclude = []
			for (let m in moodz) {
				for (let t in tensez) {
					if (verb[moodz[m]][tensez[t]]) {
						for (let f in formz) {
							if (verb[moodz[m]][tensez[t]]["forms"][formz[f]]
								&& !exclude.includes(formz[f])) {
								collection = [...collection,{verb:v,
												english:verb[moodz[m]][tensez[t]].english,
												solution:verb[moodz[m]][tensez[t]]["forms"][formz[f]],
												mood:moodz[m],
												tense:tensez[t],
												form:formz[f],
								}]
								if (["gerund", "past participle"].includes(formz[f]))
									exclude.push(formz[f])
							}
						}
					}
				}
			}
		}
		return collection
		console.log("colls", collection)
	}
	function assemble () {
		solvables = []
		solvable_count = (amount > collection.length && collection.length !== 0) ? collection.length : amount
		for (let n=0;n<solvable_count;n++) {
			const current_solvable = collection.splice(Math.floor(Math.random()*collection.length), 1)
			current_solvable[0].id = n
			console.log("currs", current_solvable)
			solvables = [...solvables.concat(current_solvable)]
		}
	}
</script>
<img src="img/success.png" class="success" class:active alt="success!!!" />
<h1>Welcome to Spanish Conjugation Practice!</h1>
<div id="main">
<div style="grid-area:block1">
	<div>
	<h2>1. Select moods</h2>
	<p>Choose what moods to include in your training!</p>
	{#each moods as m}
		<label>
			<input type="checkbox" on:change={savePrefs} bind:group={moodz} value={m} /> {m}
		</label>
	{/each}
	</div>
	<div>
	<h2>2. Select forms</h2>
	<p>Choose what verb forms to include!</p>
	{#each forms as f}
		<label>
			<input type="checkbox" on:change={savePrefs} bind:group={formz} value={f} /> {f}
		</label>
	{/each}
	</div>
</div>
<div style="grid-area:block2">
	<div>
		<h2>3. How many to solve?</h2>
		<p>How many random excercises would you like to solve?</p>
		<input type="range" on:change={savePrefs} bind:value={amount} min=1 max=20> {amount}
	</div>
	<div>
	<h2>4. Select tenses</h2>
	<p>Select all the tenses, verb forms to include in the excercise!</p>
	{#each tenses as t}
		<label>
			<input type="checkbox" on:change={savePrefs} bind:group={tensez} value={t} /> {t}
		</label>
	{/each}
	</div>
</div>
<div style="grid-area:block3">
	<div>
	<h2>5. Choose your verb!</h2>
	<p>Input one (or more) verbs to base your training on!</p>
	<input type="text" bind:value={search_term} on:keydown={handleSubmit} placeholder="verb list (comma separated)" />
	<button on:click={()=>search(search_term)}>Search</button> <div class="base_verb">{base_verb}</div>
	</div>
	<div>
	{#if solvables.length > 0}
		<p>Based on the definition, enter the correct verb form! If you guess right, the box will turn green. If you need help, click or hover on the black box for the correct solution.</p>
		<div class="solvables">
			{#each solvables as c}
				<Solvable question={c} />
			{:else}
				{#if searched}
					<p>Sorry, no matches found. Refine your search parameters!</p>
				{:else}
					<p></p>
				{/if}
			{/each}
		</div>
	{/if}
	</div>
</div>
</div>
<svelte:window on:keydown={handleSubmit}/>
<style>
	h1 {
		color: purple;
		text-align: center;
	}
	p {
		background-color: #ddd;
		padding: 4px;
	}
	#main {
		display: grid;
		grid-template-columns: 1fr 1fr 2fr;
		grid-gap: 5px;
		grid-template-areas: "block1 block2 block3";
	}
	@media all and (max-width:1000px) {
		#main {
			grid-template-columns: 1fr 1fr;
			grid-template-areas:   "block1 block2" "block3 block3";

		}
	}
	.success {
		position: fixed;
		top: 50%;
		left: 50%;
		width: 1px;
		height: 1px;
		transition: transform 2s;
	}
	.success.active {
		transform: rotate(3600deg) scale(300);
	}
	.solvables {
		display: grid;
		grid-template-columns: auto auto auto;
		justify-items: start;
		align-items: center;
	}
	.base_verb {
		display: inline-block;
		border: 1px solid black;
		padding: 6px;
		margin-left: 30px;
		vertical-align: top;
		min-width: 50px;
		text-align: center;
	}
	#main>div {
		border: 1px solid black;
		padding: 5px;
	}
	
	input[type=range]::-ms-track {
	width: 250px;height: 10px;
	background: transparent;
	border-color: transparent;
	border-width: 6px 0;
	color: transparent;
	}

	input[type=range]::-ms-fill-upper {
	background: gray;
	border-radius: 6px;
	}
	input[type=range]::-ms-fill-lower {
	background: gray;
	border-radius: 6px;
	}
</style>
