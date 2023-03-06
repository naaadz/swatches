<template>
	<div class="colors flex flex-col gap-2 h-screen">
		<form
			class="form flex flex-col p-6 m-4 w-1/2 md:w-80 md:p-10 md:m-10"
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
                :class="{ disabled : state.loading }"
			>
				Submit
			</button>
		</form>

		<template v-if="storedColors.length === 0">
			<div class="status text-3xl text-gray-300 text-center p-6 m-auto">
                <span v-if="state.loading">Loading...</span>
                <span v-else>No swatches here, go ahead and generate some.</span>
            </div>
		</template>
		<template v-else>
			<Color
				v-for="(color, i) in storedColors"
				:key="i"
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

//global vars

const increment = 6
const url = "https://www.thecolorapi.com/id"
let checkPoints = []

//reactive state

const state = reactive({
	storage: [],
	form: { s: undefined, l: undefined, showMsg: undefined },
    loading: false
})

//computed

const storedColors = computed(() => {
    //return the stored state array to the template
	return storageIndex.value !== -1 ? state.storage[storageIndex.value].colors : []
})

const storageIndex = computed(() => {
	return state.storage.findIndex((combo) => state.form.s === combo.s && state.form.l === combo.l)
})

const isStored = computed(() => {
	return state.storage.some((combo) => state.form.s === combo.s && state.form.l === combo.l)
})

//methods

const submitColors = async (e) => {
    //get the user inputs
	let s = e.target[0].value
	let l = e.target[1].value

	resetState()

	//validate user entries
	if (![s, l].every((input) => validateForm(input))) {
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
        state.loading = true
		await fetchCheckPoints()
		await fetchColors()
        state.loading = false
	}
}

const validateForm = (input) => {
	if (!input || isNaN(input) || input < 0 || input > 100) {
        state.form.showMsg = true
		return false
	}

    state.form.showMsg = false
	return true
}

const fetchCheckPoints = async () => {
	//initially create an array of colors to use as checkpoints (to sample the whole). 
    //jump to colors with the same name to cut down on requests
	for (let i = 0; i < 360; i = i + increment) {
		const res = await fetch(`${url}?hsl=${i},${state.form.s}%,${state.form.l}%`)
		const data = await res.json()
		checkPoints.push({
            name: data.name.value,
            rgb: data.rgb.value,
            contrast: data.contrast.value
		})
	}
}

const fetchColors = async () => {
    //if samples match eachother, skip ahead
	//color = always current color obj, initial will always be first checkpoint color
	let color = checkPoints[0]
	state.storage[storageIndex.value].colors.push(color)

	//x is the checkpoint counter
	let x = 0
	for (const check of checkPoints) {
		if (check.name !== color.name) {
			//go count up hues by 1, til a new color is found
			for (let i = 0; i < increment; i++) {
                //i is the hue counter, here's a magic formula
                let hslRequest = (x - 1) * increment + (i + 1)
				const res = await fetch(`${url}?hsl=${hslRequest},${state.form.s}%,${state.form.l}%`)
				const data = await res.json()
				const resColor = {
					name: data.name.value,
					rgb: data.rgb.value,
					contrast: data.contrast.value,
				}

				if (resColor.name !== color.name) {
                    //if there's a color that is between steps, set & store the color
					color = resColor
					state.storage[storageIndex.value].colors.push(color)
				}

				if (check.name === color.name) {
                    //if the color matche the next checkpoint, skip to it!
					color = check
					break
				}

				//debugger;
			}
		} else {
			//the name matches, so jump the color to the next checkpoint
			color = check
		}

		x++
	}
}

const resetState = () => {
	checkPoints = []
}

</script>
