<template>
	<div class="colors flex flex-col gap-2 h-screen">
		<form
			class="form flex bottom-0 right-0 flex-col p-6 m-4 w-2/5 md:w-80 md:p-10 md:m-10"
			@submit.prevent="submitColors"
		>
			<h1 class="text-xl md:text-3xl">Color Swatches</h1>
			<p class="hidden md:block">Enter a saturation and lightness to view swatches of available named hues.</p>
			<input
				type="text"
				class=""
				placeholder="Saturation"
			/>
			<input
				type="text"
				placeholder="Lightness"
			/>
            <p v-if="state.form.showMsg" class="text-red-500">Invalid input, try again.</p>
			<button
				type="submit"
				class="bg-gray-100 p-2"
			>
				Submit
			</button>
		</form>

		<template v-if="storedColors.length === 0">
			<p class="status text-3xl text-gray-300 text-center p-6">No swatches here, go ahead and generate some.</p>
		</template>
		<template v-else>
			<Color
				v-for="color in storedColors"
				:key="color.name"
				:name="color.name"
				:textColor="color.contrast"
				:bgColor="color.rgb"
			/>
		</template>
	</div>
</template>

<script setup>
import { reactive, computed } from "vue"
import Color from "./components/Color.vue"

const increment = 10
const url = "https://www.thecolorapi.com/id"
let checkPoints = []

const state = reactive({
	storage: [],
	form: { s: undefined, l: undefined, showMsg: undefined },
})

const submitColors = async (e) => {
	let s = e.target[0].value
	let l = e.target[1].value

	resetState()

	//validate entries
	if (![s, l].every((input) => validateForm(input))) {
		//its invalid
		return false
	}

	//set input state
	state.form.s = s
	state.form.l = l

	//if not stored, go fetch, else it will use stored values
	if (!isStored.value) {
		//setup storage
		state.storage.push({ s: state.form.s, l: state.form.l, colors: [] })
		//create the checkpoints, then make all the requests
		console.log("not stored.. go fetch")
		await fetchCheckPoints()
		fetchColors()
	}

	console.log("swatch count", storedColors.value.length)
}

const validateForm = (input) => {
    const number = parseFloat(input)

	if (!input || isNaN(number) || number < 0 || number > 100) {
        state.form.showMsg = true
		return false
	}

    state.form.showMsg = false
	return true
}

const storedColors = computed(() => {
	console.log(storageIndex.value)
	return storageIndex.value !== -1 ? state.storage[storageIndex.value].colors : []
})

const storageIndex = computed(() => {
	return state.storage.findIndex((combo) => state.form.s === combo.s && state.form.l === combo.l)
})

const isStored = computed(() => {
	return state.storage.some((combo) => state.form.s === combo.s && state.form.l === combo.l)
})

const fetchCheckPoints = async () => {
	//initially create an array of colors to use as checkpoints
	for (let i = 0; i < 360; i = i + increment) {
		const res = await fetch(`${url}?hsl=${i},${state.form.s}%,${state.form.l}%`)
		const data = await res.json()
		checkPoints.push({
			hue: data.hsl.h,
			name: data.name.value,
			rgb: data.rgb.value,
			contrast: data.contrast.value,
		})
	}
}

const fetchColors = async () => {
	//this holds the always current color, initial will always be first checkpoint color
	let color = checkPoints[0]
	state.storage[storageIndex.value].colors.push(color)

	console.log("checkPoints", checkPoints)
	let x = 0
	for (const check of checkPoints) {
		if (check.name !== color.name) {
			//go count up hues by 1
			for (let i = 0; i < increment; i++) {
				const res = await fetch(`${url}?hsl=${(x - 1) * increment + (i + 1)},${state.form.s}%,${state.form.l}%`)
				const data = await res.json()
				const resColor = {
					hue: data.hsl.h,
					name: data.name.value,
					rgb: data.rgb.value,
					contrast: data.contrast.value,
				}

				if (resColor.name !== color.name) {
					color = resColor
					state.storage[storageIndex.value].colors.push(color)
				}

				if (check.name === color.name) {
					color = check
					break
				}

				//debugger;
			}
			console.log("loop starts here")
		} else {
			//jump the hue to the next check
			console.log("x same color", x, color)
			color = check
		}

		x++
	}
}

const resetState = () => {
	checkPoints = []
}

</script>
