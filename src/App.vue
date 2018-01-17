<template>
  <div id='app'>
  <img src='./assets/tsimulus_logo.png' width='150'>
  <img src='./assets/logo_cetic.png' width='400'>
	 
	 <h2>Describe your time serie</h2>
	 
	 <ul>
	 <json-editor ref='editor' :onChange='onChange' :onModeChange='onModeChange' :json='json' style='display:inline-block;margin-right:10px;'/>
	 <json-editor-code ref='editor' :onChange='onChange' :onModeChange='onModeChange' :json='json' style='display:inline-block;'/>
	 </ul>
	
	<div id='onSubmit' style='display:inline-block;margin-right:10px;'>
	  <button v-on:click='onSubmit'>Send request</button>
	</div>
    <div id='onPlay' style='display:inline-block;margin-right:10px;'>
	  <button v-on:click='onPlay'>Play</button>
	</div>
	<div id='onPause' style='display:inline-block;margin-right:10px;'>
	  <button v-on:click='onPause'>Pause</button>
	</div>
	<div id='onStop' style='display:inline-block;margin-right:10px;'>
	  <button v-on:click='onStop'>Stop</button>
	</div>
	 <grafana/>
	 <links/>
	 {{time()}}
	 {{pause}}
     {{timer1sec()}}
  </div>
</template>

<script>
import JsonEditor from './components/JsonEditor'
import JsonEditorCode from './components/JsonEditorCode'
import Grafana from './components/Grafana'
import Links from './components/Links'
import moment from 'moment'
import Papa from 'papaparse'

export default {
  name: 'app',
  mode: 'tree',
  modes: ['code', 'form', 'text', 'tree', 'view'], // allowed modes
  data () {
    return {
      datenow: moment(Date.now()).format('YYYY-MM-DDTHH:mm:ss.SSS'),
      urlTsimulus: 'http://localhost:8001/generator/',
      urlServerNodejs: 'http://localhost:3001/sendToInflux',
      urlServerCleanUpDB: 'http://localhost:3001/cleanUpDB',
      pause: true,
      message: 'test',
      database: 'tsimulus',
      i: 0,
      lenght: 0,
      timeserie: {
        'serie': 'test',
        'value': 12.37187054752697
      },
      json: {
        'generators': [
          {
            'name': 'daily-generator',
            'type': 'daily',
            'points': {'10:00:00.000': 4, '17:00:00.000': 32}
          },
          {
            'name': 'noisy-daily',
            'type': 'aggregate',
            'aggregator': 'sum',
            'generators': [
              'daily-generator',
              {
                'type': 'arma',
                'model': {'phi': [0.5], 'std': 0.25, 'c': 0, 'seed': 159357},
                'timestep': 180000,
                'origin': '2016-01-01 12:34:56.789'
              }
            ]
          },
          {
            'name': 'partial-daily',
            'type': 'partial',
            'generator': 'daily-generator',
            'from': '2016-01-01 00:00:00.000',
            'to': '2017-01-01 00:00:00.000'
          }
        ],
        'exported': [
          {
            'name': 'series-A',
            'generator': 'daily-generator',
            'frequency': 60000
          },
          {
            'name': 'series-B',
            'generator': 'noisy-daily',
            'frequency': 30000
          }
        ],
        'from': '2016-01-01 00:00:00.000',
        'to': '2016-10-01 00:00:00.000'
      }
    }
  },
  methods: {
    onChange (newVal) {
      console.log(newVal)
      this.json = newVal
    },
    onEditable (newVal) {
      console.log(newVal)
    },
    onModeChange (newMode, oldMode) {
      console.log('Mode switched from', oldMode, 'to', newMode)
    },
    time () {
      var self = this
      this.datenow = moment(Date.now()).format('YYYY-MM-DDTHH:mm:ss.SSS')
      setTimeout(self.time, 1000)
    },
    timer1sec () {
      var self = this
      this.sendRequest()
      setTimeout(self.timer1sec, 1000)
    },
    sendRequest () {
      console.log('hello')
      if (!this.pause) {
        this.$http.post(this.urlTsimulus + this.datenow, JSON.stringify(this.json), {
          headers: {
            Accept: 'text/csv'
          }
        }).then(response => {
          console.log(response.body)
          this.message = Papa.parse(response.body)
          this.length = this.message.data.length - 1
          for (this.i = 0; this.i < this.length; this.i ++) {
            this.$http.post(this.urlServerNodejs, {'serie': this.message.data[this.i][1], 'value': this.message.data[this.i][2]}, {
              headers: {
                Accept: 'application/json'
              }
            }).then(response => {
              console.log(response.data)
            }, response => {
              console.log(response.data)
            })
          }
          this.i = 0
          this.length = 0
        }, response => {
          console.log(response.data)
        })
      }
    },
    onSubmit: function (e) {
      this.$http.post(this.urlTsimulus + this.datenow, JSON.stringify(this.json), {
        headers: {
          Accept: 'text/csv'
        }
      }).then(response => {
        console.log(response.body)
        this.message = Papa.parse(response.body)
        this.length = this.message.data.length - 1
        for (this.i = 0; this.i < this.length; this.i ++) {
          this.$http.post(this.urlServerNodejs, {'serie': this.message.data[this.i][1], 'value': this.message.data[this.i][2]}, {
            headers: {
              Accept: 'application/json'
            }
          }).then(response => {
            console.log(response.data)
          }, response => {
            console.log(response.data)
          })
        }
        this.i = 0
        this.lenght = 0
      }, response => {
        console.log(response.data)
      })
    },
    onStop: function (e) {
      this.pause = true
      this.$http.post(this.urlServerCleanUpDB, {database: this.database}).then(response => {
        console.log(response.data)
        response.status
        response.statusText
        response.headers.get('Expires')
        this.message = response.body
      }, response => {
        console.log(response.data)
      })
    },
    onPause: function (e) {
      this.pause = true
    },
    onPlay: function (e) {
      this.pause = false
    }
  },
  filters: {
    moment: function (date) {
      return moment(date).format('YYYY-MM-DDTHH:mm:ss.SSS')
    }
  },
  components: {
    JsonEditor,
    JsonEditorCode,
    Grafana,
    Links
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 40px;
}

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
</style>
