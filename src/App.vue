<template>
  <div id="app">
    <h1>Vue Template Explorer</h1>
    <label class="with-toggle"><input type="checkbox" v-model="stripWith"> Strip with?</label>
    <codemirror :code="input" :options="editorOptions" @changed="compile"></codemirror>
    <codemirror :code="output" :options="outputOptions"></codemirror>
    <transition>
      <pre class="error" v-if="errors.length"><div v-for="e in errors">{{e}}</div></pre>
    </transition>
  </div>
</template>

<script>
import { compile } from 'vue-template-compiler/build.js'
import { codemirror } from 'vue-codemirror'
import debounce from 'lodash.debounce'
import beautify from 'js-beautify'
import stripWith from 'vue-template-es2015-compiler'

function _compile (vm, input) {
  vm.input = input
  window.location.hash = encodeURIComponent(input)
  const res = compile(input, { preserveWhitespace: false })
  vm.errors = res.errors
  if (!res.errors.length) {
    let code = `function render () {${res.render.toString()}}`
    if (vm.stripWith) {
      code = stripWith(code)
    }
    vm.output = beautify(code, {
      indent_size: 2
    })
  }
}

export default {
  name: 'app',
  components: { codemirror },
  data () {
    return {
      input: '',
      output: '',
      errors: [],
      stripWith: false,
      editorOptions: {
        tabSize: 2,
        mode: 'text/html',
        theme: 'base16-light',
        lineNumbers: true,
        line: true
      },
      outputOptions: {
        tabSize: 2,
        mode: 'text/javascript',
        theme: 'base16-dark'
      }
    }
  },
  watch: {
    stripWith () {
      _compile(this, this.input)
    }
  },
  mounted () {
    const hashInput = window.location.hash.slice(1)
    if (hashInput) {
      _compile(this, decodeURIComponent(hashInput))
    }
  },
  methods: {
    compile: debounce(function (input) {
      _compile(this, input)
    }, 300)
  }
}
</script>

<style>
body {
  font-family: Menlo;
  color: #333;
  margin: 0;
}

h1 {
  padding: 0 10px;
  font-size: 1.2em;
}

.CodeMirror {
  display: inline-block;
  box-sizing: border-box;
  width: 50%;
  height: 750px !important;
  font-size: 1.1em;
  line-height: 1.3em;
}

.error {
  position: fixed;
  z-index: 999;
  margin: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #f33;
  font-size: .9em;
  color: #fff;
  font-family: Menlo;
  padding: 1em;
  transition: all .2s ease-in-out;
}

.error.v-enter, .error.v-leave-active {
  opacity: 0;
  transform: translate(0, 100%);
}

.with-toggle {
  position: fixed;
  top: 1em;
  right: 15px;
}
</style>
