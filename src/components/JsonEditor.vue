<template>

<div class="tsimulus">
   
<!--<h3>Load and save JSON documents</h3>
	<p>
  Load a JSON document: <input type="file" id="loadDocument" value="Load"/>
</p>
<p>
  Save a JSON document: <input type="button" id="saveDocument" value="Save" />
</p>-->
	
  <div v-model="json" class="editor" ref="jsoneditor" style="width:700px;height:300px;">
	</div>

	</div>
</template>

<script>
import JSONEditor from 'jsoneditor/dist/jsoneditor-minimalist.js'
import 'jsoneditor/dist/jsoneditor.min.css'
import _ from 'lodash'

export default {
  name: 'json-editor',
  data () {
    return {
      editor: null
    }
  },
  props: {
    json: {
      required: true
    },
    options: {
      type: Object,
      default: () => {
        return {}
      }
    },
    onChange: {
      type: Function
    },
    onModeChange: {
      type: Function
    }
  },
  watch: {
    json: {
      handler (newJson) {
        if (this.editor) {
          this.editor.set(newJson)
        }
      },
      deep: true
    }
  },
  methods: {
    _onChange (e) {
      if (this.onChange && this.editor) {
        this.onChange(this.editor.get())
      }
    },
    _onModeChange (e) {
      if (this.onModeChange && this.editor) {
        this.onModeChange(this.editor.get())
      }
    }
  },
  mounted () {
    const container = this.$refs.jsoneditor
    const options = _.extend({
      onChange: this._onChange
    }, this.options)

    this.editor = new JSONEditor(container, options)
    this.editor.set(this.json)
    this.editor.setMode('tree')
  },
  beforeDestroy () {
    if (this.editor) {
      this.editor.destroy()
      this.editor = null
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 15px;
}

a {
  color: #42b983;
}

#json-editor {
  width: 500px;
  height: 500px;
}

</style>
