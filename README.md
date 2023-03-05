# Swatches!

## Overview

### Performance features
- Cuts down the api requests by a (mostly) sizable amount
- Stores unique colors for each `s` and `l` combination in application state, so it doesn't fetch the api again for that input combo

### What i didn't get to do :(
- No error handling if the api is unavailable or something is skipped, the user will just see 'Loading'

## How it works & notes
The performance part was hard for me to figure out, not sure this is an acceptable solution but essentially it works like..
It does all requests at once, but the calls are limited to about 1/3 of what they would be if every possible hue was checked.

1. Setup a loop for 360 possible hues
1. Sampled those.. by grabbing every 10th color
1. Compared the samples to eachother, so if `sample[0]` has the same name as `sample[1]`, i can skip 10
1. If not the same name, go back to the last known match and go one by one til a new color is found

I thought for sure it had something to do with `closest_named_hex`, but i couldn't get it to retain the s&l 
**I am dying to know how this is supposed to be done, but i learned a ton. and got more comfortable using async functions!**

## Preview the project
This project is written in Vue3 with composition api, with Tailwind for styling.
I hosted it here for quick viewing: https://nc-swatches.netlify.app/

### Project Setup
```sh
npm install
```

#### Compile and Hot-Reload for Development
```sh
npm run dev
```

#### Compile and Minify for Production
```sh
npm run build
```
### Recommended IDE Setup
[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

### Customize configuration
See [Vite Configuration Reference](https://vitejs.dev/config/).
