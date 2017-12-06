<template>
  <div class="quill-editor">
    <slot name="toolbar"></slot>
    <div ref="editor"></div>
  </div>
</template>

<script>
  // require sources
  import _Quill from "quill"
  import defaultOptions from "./options"
  const Quill = window.Quill || _Quill

  // pollfill
  if (typeof Object.assign != "function") {
    Object.defineProperty(Object, "assign", {
      value(target, varArgs) {
        if (target == null) {
          throw new TypeError("Cannot convert undefined or null to object")
        }
        const to = Object(target)
        for (let index = 1; index < arguments.length; index++) {
          const nextSource = arguments[index]
          if (nextSource != null) {
            for (const nextKey in nextSource) {
              if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
                to[nextKey] = nextSource[nextKey]
              }
            }
          }
        }
        return to
      },
      writable: true,
      configurable: true
    })
  }

  // export
  export default {
    name: "quill-editor",
    data() {
      return {
        _options: {},
        _content: "",
        defaultOptions
      }
    },
    props: {
      content: String,
      value: String,
      disabled: Boolean,
      options: {
        type: Object,
        required: false,
        default: () => ({})
      },
      globalOptions: {
        type: Object,
        required: false,
        default: () => ({})
      }
    },
    mounted() {
      this.initialize()
    },
    beforeDestroy() {
      this.quill = null
      delete this.quill
    },
    methods: {
      // Init Quill instance
      initialize() {
        if (this.$el) {
          let icons = Quill.import("ui/icons")
          icons["bold"] = '<i class="ql-bold"></i>'
          icons["italic"] = '<i class="ql-italic"></i>'
          icons["underline"] = '<i class="ql-underline"></i>'
          icons["strike"] = '<i class="ql-strike"></i>'
          icons["blockquote"] = '<i class="ql-blockquote"></i>'
          icons["code"] = '<i class="ql-code"></i>'
          icons["code-block"] = '<i class="ql-code-block"></i>'
          icons["header"]["1"] = '<i class="ql-header1"></i>'
          // icons["header"]["2"] = '<i class="ql-header2"></i>'
          icons["header"]["2"] = '<i class="ql-header2"></i>'
          icons["header"]["3"] = '<i class="ql-header3"></i>'
          icons["header"]["4"] = '<i class="ql-header4"></i>'
          icons["header"]["5"] = '<i class="ql-header5"></i>'
          icons["list"]["ordered"] = '<i class="ql-list-ordered"></i>'
          icons["list"]["bullet"] = '<i class="ql-list-bullet"></i>'
          icons["indent"]["+1"] = '<i class="ql-indent-increase"></i>'
          icons["indent"]["-1"] = '<i class="ql-indent-decrease"></i>'
          icons["color"] = '<i class="ql-color"></i>'
          icons["background"] = '<i class="ql-background"></i>'
          icons["script"]["sub"] = '<i class="ql-script-sub"></i>'
          icons["script"]["super"] = '<i class="ql-script-super"></i>'
          icons["align"][""] = '<i class="ql-paragraph-left"></i>'
          icons["align"]["right"] = '<i class="ql-paragraph-right"></i>'
          icons["align"]["center"] = '<i class="ql-paragraph-center"></i>'
          icons["align"]["justify"] = '<i class="ql-paragraph-justify"></i>'
          icons["clean"] = '<i class="ql-clean"></i>'
          icons["link"] = '<i class="ql-link"></i>'
          icons["image"] = '<i class="ql-image"></i>'
          // Options
          this._options = Object.assign(
            {},
            this.defaultOptions,
            this.globalOptions,
            this.options
          )

          // Instance
          this.quill = new Quill(this.$refs.editor, this._options)

          // Set editor content
          if (this.value || this.content) {
            this.quill.pasteHTML(this.value || this.content)
          }

          // Disabled editor
          if (this.disabled) {
            this.quill.enable(false)
          }

          // Mark model as touched if editor lost focus
          this.quill.on("selection-change", range => {
            if (!range) {
              this.$emit("blur", this.quill)
            } else {
              this.$emit("focus", this.quill)
            }
          })

          // Update model if text changes
          this.quill.on("text-change", (delta, oldDelta, source) => {
            let html = this.$refs.editor.children[0].innerHTML
            const quill = this.quill
            const text = this.quill.getText()
            if (html === "<p><br></p>") html = ""
            this._content = html
            this.$emit("input", this._content)
            this.$emit("change", { html, text, quill })
          })

          // Emit ready event
          this.$emit("ready", this.quill)
        }
      }
    },
    watch: {
      // Watch content change
      content(newVal, oldVal) {
        if (this.quill) {
          if (newVal && newVal !== this._content) {
            this._content = newVal
            this.quill.pasteHTML(newVal)
          } else if (!newVal) {
            this.quill.setText("")
          }
        }
      },
      // Watch content change
      value(newVal, oldVal) {
        if (this.quill) {
          if (newVal && newVal !== this._content) {
            this._content = newVal
            this.quill.pasteHTML(newVal)
          } else if (!newVal) {
            this.quill.setText("")
          }
        }
      },
      // Watch disabled change
      disabled(newVal, oldVal) {
        if (this.quill) {
          this.quill.enable(!newVal)
        }
      }
    }
  }
</script>
