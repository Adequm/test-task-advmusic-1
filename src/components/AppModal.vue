<template>
	<dialog 
		ref="modal"
		class="modal__container"
		:class="{ 'modal__container-fullscreen': fullscreen }"
	>
		<div class="modal__header">
			<slot name="header"/>
			<div 
				v-if="!hideCloseButton"
				class="modal__button-close"
				@click="close"
			/>
		</div>

		<div class="modal__content">
			<slot/>
		</div>
	</dialog>
</template>

<script>

export default {
	name: 'AppModal',

	props: {
		fullscreen: Boolean,
		hideCloseButton: Boolean,
	},

	data: () => ({
		modal: null,
	}),

	mounted() {
		this.modal = this.$refs.modal;
	},

	methods: {
		open() {
			this.modal.showModal();
		},
		close() {
			this.modal.close();
		},
	},
};
</script>

<style lang="scss" scoper>
.modal {
	&__container {
		padding: 0;
		box-sizing: border-box;
		overflow: hidden;
		position: relative;
		&-fullscreen {
			width: 100%;
		}
		&::backdrop {
			background-color: #0008;
			backdrop-filter: blur(2px);
		}	
		.modal__header {
			position: absolute;
			z-index: 1;
			top: 0;
			right: 0;
			width: 100%;
			min-height: 50px;
	    display: flex;
	    justify-content: flex-end;
	    align-items: center;
	    padding: 10px;
		}
		.modal__content {
			line-height: 1;
	    margin-bottom: -3px;
	    background: #000;
		}
	}
	&__button-close {
		width: 30px;
		height: 30px;
		cursor: pointer;
		color: #fff;
		z-index: 2;
		display: flex;
		justify-content: center;
		align-items: center;
		&::after, &::before {
			content: "";
			position: absolute;
			width: 4px;
			height: inherit;
			border-radius: 2px;
			background: currentColor;
		}
		&::before { transform: rotate(45deg); }
		&::after { transform: rotate(-45deg); }
	}
}
</style>