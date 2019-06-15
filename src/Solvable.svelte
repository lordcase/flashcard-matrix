<script>

import { onMount } from 'svelte';

export let question
let answer = ''
let correct = false
let unhidden = false
function verify() {
    correct = answer.toUpperCase() === question.solution.toUpperCase()
}
onMount(()=>{
    console.log("mounted - " + question.solution)
    return ()=>{console.log("unmmm")}
    })

</script>


<div>{question.verb} - {question.mood}-{question.tense}-{question.form} </div><div><input type="text" bind:value={answer} on:keyup={verify} data-stop="true" class:correct /></div> <div class="hidden" class:unhidden on:click="{() => unhidden = !unhidden}">{question.solution}</div>
<style>
input.correct {
    background-color: green;
    color: white;
    
}
.hidden {
    color: black;
    background:black;
    display: inline-block;
    align-self:center;
    padding: 6px;
}
.hidden:hover{color:white}
.hidden.unhidden{color:white}
</style>