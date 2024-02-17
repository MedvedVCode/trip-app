<template>
	<div
		v-if="isOpen"
		class="modal"
	>
		<input
			type="text"
			min="2"
			class="modal__input"
			v-model="userName"
		/>
		<button
			class="round-btn"
			@click="addUser"
		>
			&#10004;
		</button>
		<button
			class="round-btn"
			@click="closeModal"
		>
			&times;
		</button>
	</div>
</template>

<script>
export default {
	name: 'ModalUserAdd',
	data() {
		return {
			isOpen: false,
			userName: '',
		};
	},
	methods: {
		showModal() {
			this.isOpen = true;
		},
		closeModal() {
			this.userName = '';
			this.isOpen = false;
		},
		addUser() {
			this.$emit('newUser', this.userName);
			this.userName = '';
		},
	},
};
</script>

<style lang="scss" scoped>
.modal {
	margin-top: 10px;
	display: flex;
	justify-content: center;
	gap: 10px;
	@include jost(16px, 400, 29px, 0.16px);
	&__input{
		padding-left: 5px;
		border-radius: 5px;
		border: 2px solid $accentColor;
		color: $textColor;
		max-width: 160px;
	}
}
</style>
