<script setup lang="ts">
import NumbersNew from './components/NumbersNew.vue';
</script>

<template>
	<!-- There is already a id attribute in index.html -->
	<!-- <div id="app"> -->
	<div>
		<NumbersNew :start-limit="100" :max-limit="10000"  />
	</div>
</template>
