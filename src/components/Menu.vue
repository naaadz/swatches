<template>
    <div class="menu absolute">
        <form
			class="flex flex-col space-y-4"
            @submit.prevent="submitColors"
		>
			<input
				type="text"
				placeholder="Saturation"
                :value="state.form.s"
			/>
			<input
				type="text"
				placeholder="Lightness"
                :value="state.form.l"
			/>
            <button type="submit">Submit</button>
		</form>
    </div>
</template>

<script setup>

defineProps({
    name: {
        type: String,
		required: true
    },
	textColor: {
		type: String,
		required: true
	},
    bgColor: {
		type: String,
		required: true
	},
})
</script>

<style>

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

</style>
