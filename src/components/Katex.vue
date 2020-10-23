<template>
  <div class="cont">
    <div id="input-cont">
      <div id="color" :class="{night}" v-html="color"></div>
      <textarea v-model="code" id="input" :class="{night}" placeholder="Write math here" spellcheck="false"></textarea>
    </div>
    
    <div v-katex="{expression: text, options: {throwOnError: false, displayMode: true}}" :class="{night}" id="katex" v-show="latex"></div>
    <div id="katex" :class="{night}" v-show="!latex">{{text}}</div>
    <button :class="{btn: true, 'btn-latex': true, night}" @click="showLatex" v-show="latex">Show Text</button>
    <button :class="{btn: true, 'btn-latex': true, night}" @click="showLatex" v-show="!latex">Show Render</button>
    <button :class="{btn: true, 'btn-night': true, night}" @click="nightMode" v-show="!night">Dark Mode</button>
    <button :class="{btn: true, 'btn-night': true, night}" @click="nightMode" v-show="night">Light Mode</button>
  </div>
</template>

<script>

const LEN = 4

export default {
  name: 'Katex',
  data() {
    return {
      code: '\'Use code-like syntax to quickly write math equasions\'\\\\\n' +
            '/{a^2 + b^2}{c^2}\\\\\n' +
            '\'Use doube backslash to break line\'\\\\\n' + 
            '"Logical operators"\\\\\n' + 
            'p && q => r || s <=> !p && !q \\\\\n' +
            '"And any basic LaTeX expression"\\\\\n' +
            '\\sum_{i = 0}{n}f(i)-f(i^2)\n',
      text: '',
      color: '',
      latex: true,
      night: true
    }
  },
  methods: {
    showLatex() {
      this.latex = !this.latex
    },
    nightMode() {
      this.night = !this.night
      localStorage.setItem('night', this.night)
      this.$emit('night', this.night)
    }
  },
  mounted() {
    let night = localStorage.getItem('night')
    if (night === null || night === undefined) {
      localStorage.setItem('night', true)
    }
    else {
      this.night = night
    }
    this.$emit('night', this.night)
  },
  watch: {
    code: {
      immediate: true,
      handler() {
        this.text = ''
        let last = Array(LEN)
        let text = false
        for (const sym of this.code) {
          last[0] = last[1]
          last[1] = last[2]
          last[2] = sym
          if (/['"]/.test(last.slice(LEN - 2, LEN).join('')) && last[1] != '\\') {
            if (text) {
              text = false
              this.text += '}'
              continue
            }
            else {
              text = true
              this.text += "\\text{"
              continue
            }
          }
          if (!text) {
            if (last.slice(LEN - 4, LEN).join('') == '<=>') {
              this.text = this.text.slice(0, -4)
              this.text += "\\Leftrightarrow "
              continue
            }
            if (last.slice(LEN - 3, LEN).join('') == '=>') {
              this.text = this.text.slice(0, -2)
              this.text += "\\Rightarrow "
              continue
            }
            if (last.slice(LEN - 3, LEN).join('') == '<=') {
              this.text = this.text.slice(0, -2)
              this.text += "\\le "
              continue
            }
            if (last.slice(LEN - 3, LEN).join('') == '>=') {
              this.text = this.text.slice(0, -2)
              this.text += "\\ge "
              continue
            }
            if (last.slice(LEN - 2, LEN).join('') == '!') {
              if (last[LEN - 3] != '#') {
                this.text += "\\neg "
                continue
              }
              else this.text = this.text.slice(0, -2)
            }
            if (last.slice(LEN - 3, LEN).join('') == '/{') {
              if (last[LEN - 4] != '#') {
                this.text = this.text.slice(0, -2)
                this.text += "\\frac{"
                continue
              }
            }
            if (last.slice(LEN - 3, LEN).join('') == '&&') {
                this.text = this.text.slice(0, -2)
                this.text += "\\land "
                continue
            }
            if (last.slice(LEN - 3, LEN).join('') == '||') {
                this.text = this.text.slice(0, -2)
                this.text += "\\lor "
                continue
            }
            this.text += sym
          }
          else {
            if (sym == '{') this.text += "\\{"
            if (sym == '}') this.text += "\\}"
            else this.text += sym
          }
        }

        let latex = false
        text = false
        this.color = ''
        for (let sym of this.code) {
          last[0] = last[1]
          last[1] = last[2]
          last[2] = sym
          if (sym == '\\' && !latex) {
            latex = true
            this.color += "<purple>\\"
            continue
          }
          if (/['"]/.test(sym) && !text && last[1] != '\\') {
            text = true
            this.color += "<green>"
            this.color += sym
            continue
          }
          if (/['"]/.test(sym) && text && last[1] != '\\') {
            text = false
            this.color += sym
            this.color += "</green>"
            continue
          }
          if (sym == ' ') {
            this.color += "&nbsp;"
            continue
          }
          if (sym == '\n') {
            this.color += "<br>"
            continue
          }
          if (!text) {
            if (sym == '\\' && last[1] == '\\') {
              this.color = this.color.slice(0, -1)
              if (latex) {
                latex = false
                this.color += "</purple>"
              }
              this.color += "<gray>↵&nbsp;</gray>"
              continue
            }
            if (latex && /[^A-Za-z]/.test(sym)) {
              latex = false
              this.color += "</purple>"
            }
            if (/[0-9]/.test(sym)) {
              this.color += `<orange>${sym}</orange>`
              continue
            }
            if (/[^A-Za-z0-9]/.test(sym)) {
              this.color += `<gray>${sym}</gray>`
              continue
            }
          } 
          this.color += sym
        }
      }
    }
  }
}
</script>

<style>
  @import "~@/assets/katex.min.css";

  @font-face {
    src: url('~@/assets/hack.ttf');
    font-family: Hack;
  }
  .btn-latex {
    right: 55px;
  }
  .btn {
    top: 50px;
    position: absolute;
    background: #eee;
    border: none;
    color: #888;
    padding: 10px;
    border-radius: 10px;
    user-select: none;
    transition: 200ms;
    cursor: pointer;
  }
  .btn:hover {
    transform: scale(0.95);
    color: #555;
  }
  .btn:active {
    transform: scale(0.9);
    background: #ddd;
    color: #222;
  }
  .btn.night {
    background: #444;
    color: #999;
  }
  .btn.night:hover {
    transform: scale(0.95);
    color: #aaa;
  }
  .btn.night:active {
    transform: scale(0.9);
    background: #222;
    color: #fff;
  }
  .cont {
    display: flex;
    margin: 0 50px;
  }
  purple {
    color: #cc6bc4;
    display: inline;
  }
  orange {
    color: #ff851a;
    display: inline;
  }
  gray {
    color: #555;
    display: inline;
  }
  green {
    color: #79c534;
    display: inline;
  }

  #input {
    position: absolute;
    top: 0;
    left: 0;
    background: transparent;
    font-family: Hack;
    resize: none;
    color: #222;
    border: none;
    border-radius: 10px;
    height: 70vh;
    padding: 20px;
    font-size: 18px;
    box-shadow: 0 5px 10px #aaa;
    width: 100%;
    color: transparent;
    caret-color: black;
  }
  #input.night {
    caret-color: #aaa;
    box-shadow: 0 5px 10px #111;
  }
  #input-cont { 
    width: 50vw;
    position: relative;
    margin-right: 25px;
  }
  #color {
    background: white;
    color: black;
    position: absolute;
    left: 0;
    top: 0;
    height: 70vh;
    padding: 20px;
    width: 100%;
    font-size: 18px;
    text-align: left;
    z-index: -1;
    box-shadow: 0 5px 10px #aaa;
    border: none;
    border-radius: 10px;
    padding: 20px;
    font-family: Hack;
  }
  #color.night {
    box-shadow: 0 5px 10px #111;
    background: #222;
    color: #aaa;
  }
  #katex {
    text-align: left;
    background: white;
    color: #222;
    border-radius: 10px;
    width: 50vw;
    height: 70vh;
    padding: 20px;
    font-size: 18px;
    box-shadow: 0 5px 10px #aaa;
    margin-left: 25px
  }
  #katex.night {
    box-shadow: 0 5px 10px #111;
    background: #222;
    color: #aaa;
  }
</style>
