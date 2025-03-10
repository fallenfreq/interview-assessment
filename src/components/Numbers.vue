<script setup lang="ts">
const numbers = [];
let limit = 100;

function n()
{
	let numbers = []; 
	for(var i = 0; i < limit; i++) { numbers = [...numbers, i]; }

	return numbers.sort(() => Math.random() - 0.5);
}

function hov(number) {
  const nums = document.querySelectorAll('.number');

  for(let i = 0; i < nums.length; i++)
  {
    const num = nums[i].textContent.trim();
    if(number % num === 0)
    {
      nums[i].classList.add('active')
      console.log('divisor', num)
    }
  }
}

function reset()
{
	const nums = document.querySelectorAll('.number');
	nums.forEach(num => num.classList.remove('active'))
}
</script>

<template>
	<div>
		<input type="number" v-model="limit" /><br /><br /> 
		<div class="number"
			:id="'number-'+number"
			v-for="number in n()"
			:key="number"
			@mouseover="hov(number)"
			@mouseout="reset"
		>
			{{ number }}
		</div>
	</div>
</template>


<style scoped>
.number {
	display: inline-block;
	padding: 5px;
	background-color: lightgrey;
	margin: 5px;
}

.active {
	background-color: red;
}
</style>

