<template>
	<div>
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

    <!-- <Color 
        :textColor="color.contrast" 
        :bgColor="color.rgb" 
    /> -->

	<main>
        <ul class="flex flex-col">
            <li 
                v-for="color in storedColors" 
                class="flex"
                :key="color.hue" 
                :style="`background: ${color.rgb};`">
                <span 
                    v-for="(value,key) in color"
                    class="flex-1"
                    :style="`color: ${color.contrast}`"
                >
                        {{ key }} : {{ value }}
                </span>
            </li>
        </ul>
	</main>
</template>




<script setup>
import { reactive, computed } from 'vue'

const increment = 10
const url = 'https://www.thecolorapi.com/id'
let checkPoints = []

const state = reactive({
    storage: [],
    form: { s: 100, l: 50 }
})

const submitColors = async(e) => {
    let s = e.target[0].value
    let l = e.target[1].value

    resetState()

    //validate entries
    if (![s, l].every(input => isInputValid(input))) {
        //its invalid   
        return false
    } 

    //set input state
    state.form.s = s
    state.form.l = l


    //if not stored, go fetch, else it will use stored values
    if (!isStored.value) {
        //setup storage
        state.storage.push({ s: state.form.s, l:state.form.l, colors: []})
        //create the checkpoints, then make all the requests
        console.log('not stored.. go fetch')
        await fetchCheckPoints()
        fetchColors()
    } 

}

const isInputValid = (input) => {
    if (!input) {
        return false
    } 

    const number = parseFloat(input)
    
    if ( isNaN(number) || number < 0 || number > 100 ) {
        return false
    } 

    return true
}

const storedColors = computed(() => {
    console.log(storageIndex.value)
    return storageIndex.value !== -1 ? state.storage[storageIndex.value].colors : []
})

const storageIndex = computed(() => {
    return state.storage.findIndex(combo => state.form.s === combo.s && state.form.l === combo.l)
})

const isStored = computed(() => {
    return state.storage.some(combo => state.form.s === combo.s && state.form.l === combo.l)
})

const fetchCheckPoints = async() => {
    //initially create an array of colors to use as checkpoints
    console.log('fetchCheckPoints started')
    for (let i = 0; i < 360; i=i+increment) {
        const res = await fetch(`${url}?hsl=${i},${state.form.s}%,${state.form.l}%`)
        const data = await res.json()
        checkPoints.push({ 
            'hue': data.hsl.h, 
            'name': data.name.value, 
            'rgb': data.rgb.value,
            'contrast': data.contrast.value
        })
    }
}

const fetchColors = async() => {
    console.log('fetchColors started')
    //this holds the always current color, initial will always be first checkpoint color
    let color = checkPoints[0]
    state.storage[storageIndex.value].colors.push(color)

    console.log('checkPoints', checkPoints)
    let x = 0
    for (const check of checkPoints) {
        if (check.name !== color.name) {
            //go count up hues by 1
            for (let i = 0; i < increment; i++) {
                const res = await fetch(`${url}?hsl=${((x - 1) * increment) + (i + 1)},${state.form.s}%,${state.form.l}%`)
                const data = await res.json()
                const resColor = { 
                    'hue': data.hsl.h, 
                    'name': data.name.value, 
                    'rgb': data.rgb.value,
                    'contrast': data.contrast.value
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
            console.log('loop starts here')

        } else {
            //jump the hue to the next check
            console.log('x same color', x, color)
            color = check
        }

        x++
    }
}

const resetState = () => {
    checkPoints = []
}

</script>