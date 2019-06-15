<script>

import {correct_count} from './stores.js'

export let question
let answer = ''
let correct = false
let unhidden = false
$correct_count[question.id] = false
function verify() {
    if (answer.toUpperCase() === question.solution.toUpperCase()) {
        correct = true
        $correct_count[question.id] = true
    } else {
        correct = false
        $correct_count[question.id] = false
    }
}

</script>


<div>{#if ['gerund', 'past participle'].includes(question.form)}{question.verb} - {question.form} {:else} {question.verb} - {question.mood}-{question.tense}-{question.form} {/if}</div><div><input type="text" bind:value={answer} on:keyup={verify} data-stop="true" class:correct /></div> <div class="hidden" class:unhidden on:click="{() => unhidden = !unhidden}">{question.solution}</div>
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