<template>
    <Transition appear>
        <div class="color flex flex-1 items-center justify-left gap-3 p-3" :style="`background: ${bgColor};`">
            <span class="name" :style="`color: ${textColor}`">{{ name }}</span>
            <span class="bg-color font-mono" :style="`color: ${textColor}`">{{ bgColor }}</span>
        </div>
    </Transition>
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
