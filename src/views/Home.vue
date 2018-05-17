<template>
  	<main class="hero is-fullheight is-primary">
		<div class="hero-body">
			<div class="container">
				<div class="columns is-centered">
					<div class="column is-6 is-flex is-column is-centered">
						<div class="image keswah-logo">
							<img src="../assets/img/logo.png" alt="">
						</div>
            <Login />
					</div>
				</div>
			</div>
		</div>
	</main></template>
 
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Login from "@/components/Login.vue"; // @ is an alias to /src

@Component({
  components: {
    Login
  } 
})
export default class Home extends Vue {}
</script>
<style lang="scss">
.keswah-logo {
  max-width: 15rem;
  margin-bottom: 5rem;
}
.column.is-flex {
  &.is-column {
    flex-direction: column;
  }
  &.is-centered {
    align-items: center;
    justify-content: center;
  }
}

</style>
