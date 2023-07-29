<template>
	<n-message-provider>
		<Login />
	</n-message-provider>
	
</template>

<script>
	import Login from './components/Login.vue'

	export default {
		name: 'App',
		components: {
			Login
		},
		setup() {

		},
	};
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}
</style>
