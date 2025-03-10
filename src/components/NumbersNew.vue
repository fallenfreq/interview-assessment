<script setup lang="ts">
import { ref, type Ref, computed, watch } from 'vue'

// GENERAL NOTES:
// The original Numbers.vue component has been left for comparison and removed from the tsconfig to allow successful builds.
// lang="ts" is being used in the original, yet there are no type definitions.
// Naming conventions are unclear or ambiguous and not self-explanatory.

// `maxLimit` and `startLimit` are now configurable via props making the component more reusable and flexible.
// `props` need to be defined with the type-based Declaration provide type information.
const props = defineProps<{
  maxLimit: number
  startLimit: number
}>()

// `const numbers` is never used because `n` is creating its own array.

// OLD CODE
// const numbers = [];

// Computed properties automatically cache and update when their dependencies change,
// ensuring that the array is only reshuffled when `limit` changes.

const shuffledNumbers = computed(() => {
  return shuffle(Array.from({ length: limit.value }, (_, i) => i + 1))
})

// This is used to style the active numbers instead of manually adding and removing classes.
const divisorsOfHovered = ref(new Set<number>())

// `limit` should be a reactive property since it updates via v-model,
// ensuring that changes trigger updates in the template.

// OLD CODE
// let limit = 100

// NEW CODE
// `rawLimit` holds the raw input value as a string since v-model on an <input> typically returns a string.
// This ensures that the input field correctly reflects what the user types.
// There are more notes in the template section regarding type="number" VS type="text" and the reason for rawLimit.

// Note: In Vue, if you bind `v-model` to a `Ref<number>`, the input value will be coerced to a number 
// when valid. However, if the user types a string or an incomplete number in scientific notation (e.g., "1e"),
// the value will remain a string, potentially causing a type mismatch. To avoid this, we explicitly use 
// `Ref<string>` for `rawLimit` and handle type conversion separately.
const rawLimit: Ref<string> = ref(props.startLimit.toString())

// `limit` stores a valid number derived from `rawLimit`.
// This is used by the rest of the app to control the grid generation.
const limit: Ref<number> = ref(100)

// `errorMessage` holds validation errors separately from `limit`.
const errorMessage = ref('')

// The `watch` function ensures validation runs every time `rawLimit` changes.
// This approach separates concerns by keeping user input (`rawLimit`)
// independent from the validated value used in rendering (`limit`).

watch(rawLimit, (newValue) => {
  const parsedValue = Number(newValue)

  // Allow empty input without showing an error, but prevent grid rendering.
  if (newValue.trim() === '') {
    errorMessage.value = ''
    limit.value = 0 // Prevents grid from rendering until valid input is provided
    return
  }

  // Validate that the input is a positive integer.
  if (!Number.isInteger(parsedValue) || parsedValue < 1) {
    errorMessage.value = 'Please enter a valid positive integer.'
    limit.value = 0 // Prevents grid from rendering until valid input is provided
    return
  }

  // Setting an upper limit to prevent the user from entering a number that is too large to be computed.
  // This also helps to keep the UI more predictable since we can determine the maximum width of a number.
  if (parsedValue > props.maxLimit) {
    errorMessage.value = 'Please enter a number less than or equal to 1000.'
    limit.value = 0
  return
}

  // If valid, update `limit` and clear any existing error message.
  limit.value = parsedValue
  errorMessage.value = ''
})

// n() NOTES:
// `var` is used, which has function scope and can lead to inconsistencies.
// `let` is block-scoped and is the preferred way to declare variables in modern JavaScript.
// The use of `[...numbers, i]` causes O(n^2) time complexity.
// Using `.push()` would be more efficient at O(n).
// The function name `n` is not descriptive.
// The approach to shuffling the array is not uniformly random.
// Sort uses Timsort, which has a Θ(n log n) time complexity.
// A better approach would be to use the Fisher-Yates shuffle, which has O(n) time complexity.
// `i` starts at 0 when the scope is to use all numbers between 1 and `limit`.

// OLD FUNCTION
// function n() {
//   let numbers = []
//   for (var i = 0; i < limit.value; i++) {
//     numbers = [...numbers, i]
//   }
//   return numbers.sort(() => Math.random() - 0.5)
// }

// NEW FUNCTION
function shuffle(array: number[]): number[] {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

// `hov` NOTES:
// `nums`, `num`, and `number` are not descriptive and are confusing.
// The function name `hov` is not descriptive.
// `num` needs to be converted to a number before performing the modulus check,
// as `.textContent` returns a string.
// There would be no need to use querySelectorAll if `numbers` was accessible directly.
// There is no error handling or validation for the input.
// Instead of manually adding and removing classes, `:class` is used and the state
// of active numbers is kept in a reactive `ref`. (See `reset` NOTES: for more details)

// OLD FUNCTION
// function hov(number) {
//   const nums = document.querySelectorAll('.number')

//   for (let i = 0; i < nums.length; i++) {
//     const num = nums[i].textContent.trim()
//     if (number % num === 0) {
//       nums[i].classList.add('active')
//       console.log('divisor', num)
//     }
//   }
// }

// NEW FUNCTION
function highlightDivisors(targetNumber: number) {
  clearDivisors()
  for (const number of shuffledNumbers.value) {
    if (targetNumber % number === 0) {
      divisorsOfHovered.value.add(number)
    }
  }
}

// `reset` NOTES:
// `reset` is not descriptive enough.
// The querySelectorAll could just look for `.active` instead of all `.number` elements.
// Manually adding and removing classes can lead to inconsistencies,
// especially if multiple event listeners modify the same element.
// Using Vue’s reactivity with `:class` ensures state changes are tracked automatically.
// In a larger app, the state would likely be stored anyway and used by other aspects of the app.

// OLD FUNCTION
// function reset() {
//   const nums = document.querySelectorAll('.number')
//   nums.forEach((num) => num.classList.remove('active'))
// }

// NEW FUNCTION
function clearDivisors() {
  divisorsOfHovered.value.clear()
}

</script>

<template>
  <div>
    <!--
      `br` should not be used for spacing because it is not semantic.
      All input elements should have a label for accessibility and user instructions.
      A class has been added to the input for styling, which can replace these <br> tags.
      The input could be shown even if the value is not a valid input which can be a smoother experience for the user.
      An error message can then be displayed when the input is invalid using aria-live="polite" for accessability.
      -->

    <!-- type="number" VS type="text":
      There are trade-offs between using `type="number"` and `type="text"` for input fields.
      `number` will allow the user to type scientific notation, which will make `rawLimit` appear as an empty value when a partial number is typed.
      Empty value is valid when the input is empty so there is no way to give error feedback to the user without manually retrieving and validating the value from the dom.
      For example, 1e is an error while using `text` but valid while using `number` but behaves as though the input was empty as 1e is an incomplete number and still requires an exponent.
      Either type can be appropriate depending on the desired user experience. text is used here to show the alternative which avoids type mismatches and gives a smoother user experience.
      When the input box is longer, it can be easier for the user to miss that it only accepts numbers and can be confusing as to why typing is not working as expected.
      `Inputmode` is used to hint to the browser that the input should be a number which will open the number pad for mobile users still.
      -->
    
    <!-- OLD CODE 
     <input type="number" v-model="limit" /><br /><br />
     -->

    <!-- NEW CODE -->
    <label for="limit-input">Enter a number between 1 and {{maxLimit}}:</label>
    <input
      id="limit-input"
      type="text"
      v-model="rawLimit"
      class="limit-input"
      inputmode="numeric"
    />
    <p v-if="errorMessage" aria-live="polite" class="error-message">{{ errorMessage }}</p>

    <!--
      :class is used to conditionally add the `active` class to the `.number` divs.
      The :id binding is not used and can be removed in this case.
      Function calls such as `n()` should not be used in v-for because they will be re-executed on every render, 
      rather than being computed once and cached.
      Wrapping `.number` divs in a container improves styling predictability and flexibility, 
      enables proper grid/flexbox behavior, and avoids conflicts when using `v-if` with `v-for`.
      `v-else` is used to display the grid only when `limit` is valid,  
      though setting `limit` to `0` when the input is invalid inherently prevents rendering anyway. 
      -->

    <!-- OLD CODE 
      <div class="number"
        :id="'number-'+number"
        v-for="number in n()"
        :key="number"
        @mouseover="hov(number)"
        @mouseout="reset"
      > -->

    <!-- NEW CODE -->
    <div v-else class="grid">
      <div
        class="number"
        :class="{ active: divisorsOfHovered.has(number) }"
        v-for="number in shuffledNumbers"
        :key="number"
        @mouseover="highlightDivisors(number)"
        @mouseout="clearDivisors"
      >
        {{ number }}
      </div>
    </div>
  </div>
</template>

<!-- 
The style tag should be `scoped`—or use `module`—to the component, 
especially when using generic classes like `.number` or `.active`.
This prevents unintended styling conflicts with other components.
-->

<!-- 
There were two divs with the id of `app` in the original code.
The one in `App.vue` was removed, and the one in `index.html` was kept.
box-sizing: border-box; was added to the 'styles.css file'
A width of 100% was added to `#app` in `styles.css` so that full width can be used in this component.
-->
<style scoped>
/* Replaces the <br> tags for better structure and spacing. 
Using block display ensures full-width input and consistent spacing below. */
.limit-input {
  width: 100%;
  display: block;
  margin-bottom: 10px;
  padding: 10px;
}

/* Wraps the numbers for easier layout control.
inline-block could achieve similar and with less syntax, however, flexbox is more declarative.
It also keeps the layout adaptable for future changes (e.g., responsiveness, reordering),
which would be harder if not impossible with inline-block */
.grid {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  justify-content: center;
}

/*
Ensures all numbers in a row grow equally to fill remaining space.
This is more aesthetically pleasing and would not be possible with inline-block
*/
.number {
  display: flex;
  flex-grow: 1;
  align-items: center;
  justify-content: center;
  padding: 5px;

  /* min-width provides more consistent styling for smaller numbers. */
  /* min-width set to 50px to cover the largest content since there is a default max limit of 1000. */
  /* Larger numbers will still expand their containers if the limit is increased. */
  height: 35px;
  min-width: 50px; 
  text-align: center;

  /* `gap` from `.grid` handles spacing, so margin isn't needed. */
  /* margin: 5px; */

  /* Improves contrast for readability */
  font-weight: bold;
  background-color: rgb(91, 124, 26);
  color: #fff;

  /* Indicates interactivity */
  cursor: pointer;
}

.active {
  background-color: red;
}

.error-message {
  color: red;
  font-size: 14px;
  margin-top: 5px;
}

</style>
