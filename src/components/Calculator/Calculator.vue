<template>
  <div class="calculator">
    <div class="calculator__frame">
      <div class="calculator__display">
        <div class="calculator__logs">
          <span v-for="log of logs" class="calculator__log">{{ log }}</span>
        </div>
        <div class="calculator__sentence">{{ sentence }}</div>
      </div>

      <div class="calculator__buttons">
        <div class="calculator__button clear" @click.prevent="clear()">C</div>
        <div class="calculator__button clear" @click.prevent="del()">&#9003;</div>
        <div class="calculator__button operator" @click.prevent="addExpresion('%')">%</div>
        <div class="calculator__button operator" @click.prevent="addExpresion('/')">/</div>
        <div class="calculator__button" @click.prevent="addExpresion('7')">7</div>
        <div class="calculator__button" @click.prevent="addExpresion('8')">8</div>
        <div class="calculator__button" @click.prevent="addExpresion('9')">9</div>
        <div class="calculator__button operator" @click.prevent="addExpresion('*')">*</div>
        <div class="calculator__button" @click.prevent="addExpresion('4')">4</div>
        <div class="calculator__button" @click.prevent="addExpresion('5')">5</div>
        <div class="calculator__button" @click.prevent="addExpresion('6')">6</div>
        <div class="calculator__button operator" @click.prevent="addExpresion('-')">-</div>
        <div class="calculator__button" @click.prevent="addExpresion('1')">1</div>
        <div class="calculator__button" @click.prevent="addExpresion('2')">2</div>
        <div class="calculator__button" @click.prevent="addExpresion('3')">3</div>
        <div class="calculator__button operator" @click.prevent="addExpresion('+')">+</div>
        <div class="calculator__button" @click.prevent="addExpresion('.')">.</div>
        <div class="calculator__button" @click.prevent="addExpresion('0')">0</div>
        <div class="calculator__button wide accent" @click.prevent="getResult()">=</div>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, computed, watch } from 'vue'
  const logs = ref([])
  const sentence = ref(0)
  const showingResult = ref(true)

  const lastDigit = computed(() => sentence.value.length ? sentence.value[sentence.value.length-1] : null)

  // Clear eval sentence
  const clear = () => {
    if(Number(sentence.value) === 0) logs.value = []
    sentence.value = 0
    showingResult.value = false
  }

  // Remove last char of eval sentence
  const del = () => {
    if(Number(sentence.value) === 0) clear()
    if(String(sentence.value) === 'ERROR') clear()

    sentence.value = String(sentence.value)
    sentence.value = sentence.value.slice(0, -1)
    if(!sentence.value) sentence.value = 0
  }

  const addExpresion = (e) => {
    // Clear if error
    if(sentence.value === 'ERROR') clear()

    // Add zero before alone dot
    if(isNaN(lastDigit.value) && isNaN(e) && e === '.') e = '0.'

    // Prevent double operators
    if(isNaN(lastDigit.value) && isNaN(e) && e !== '.') del()

    // Change first number if input value is a number and we are watching an eval result
    if(!isNaN(sentence.value) && !isNaN(e) && showingResult.value) clear()

    // If we want to operate with result we reset showingResult var
    if(showingResult.value) showingResult.value = false

    // Set new value
    sentence.value += e

    // Remove left zeros if needed
    deleteZeroPad()
  }

  // Remove left zeros
  const deleteZeroPad = () => {
    if(String(sentence.value).match('^0[0-9].*$')) sentence.value = String(parseInt(sentence.value))
  }

  // Prevent eval precision decimal issues
  const precisionRound = (number, precision = 4) => {
    const factor = Math.pow(10, precision);
    return Math.round(number * factor) / factor;
  }

  // Remove extra dots
  const removeExtraDots = () => {
    if(sentence.value === 'ERROR') return
    const numericSentenceValues = String(sentence.value).match(/[\d\.]+/g)
    numericSentenceValues.forEach((dirtyNumber) => {
      if(dirtyNumber.match(/\./g)?.length > 1) del()
    })
  }

  // Make eval, show result and store log
  const getResult = () => {
    try {
      // Prevent non evaluable sentence
      const NaNs = String(sentence.value).match(/\D+/g)
      if(String(sentence.value).match(/\d+/g).length < 2 || (NaNs.length == 1 && NaNs[0] === '.')) return

      // Remove last char if it's an operator
      if(isNaN(lastDigit.value)) del()

      const oldSentence = sentence.value
      const evaluatedSentence = eval(sentence.value)
      const result = precisionRound(evaluatedSentence)

      // Prevent division by 0 error and not number errors
      if(!isFinite(result) || isNaN(result)){
        sentence.value = 'ERROR'
        return
      }

      sentence.value = result
      showingResult.value = true
      logs.value.push(`${oldSentence}=${result}` )
    } catch (e) {
      if(e instanceof SyntaxError) {
        alert(e.message);
      } else {
        throw e;
      }
    }
  }

  watch(sentence, () => removeExtraDots())
</script>

<style lang="scss">
  @import './calculator.scss';
</style>
