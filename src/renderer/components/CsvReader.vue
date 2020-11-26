<template>
  <div>
    <input type="file" @change="loadCsvFile"/>
    <p>{{ message }}</p>

    <v-row>
      <v-col cols="10">
        <v-data-table
            :headers="header"
            :items="data"
        >
          <template v-slot:item="{ item }">
            <tr>
              <td :key="i" v-for="(v, i) in item">{{ v }}</td>
            </tr>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import Encoding from 'encoding-japanese'

export default {
  data () {
    return {
      message: '',
      data: [],
      header: []
    }
  },
  methods: {
    loadCsvFile (e) {
      const vm = this
      vm.data = []
      vm.message = ''
      const file = e.target.files[0]

      if (!file.type.match('text/csv')) {
        vm.message = 'CSVファイルを選択してください'
        return
      }

      const reader = new FileReader()
      reader.onload = e => {
        const result = new Uint8Array(e.target.result)
        const encode = Encoding.detect(result)
        const shiftJisCodeList = Encoding.convert(result, {
          to: 'unicode',
          from: encode,
          type: 'string'
        })
        const lines = shiftJisCodeList.split('\n')
        lines.shift()
        const linesArr = []
        if (lines.length === 0) return

        for (let i = 0; i < lines.length; i++) {
          linesArr[i] = lines[i].split(',')
        }

        const header = []
        linesArr[0].forEach(h => header.push({
          text: h,
          value: h
        }))

        vm.header = header

        if (lines.length === 1) return
        vm.data = linesArr.splice(1)
      }
      reader.readAsArrayBuffer(file)
    }
  }
}
</script>
