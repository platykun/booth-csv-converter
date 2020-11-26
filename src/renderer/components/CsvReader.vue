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
      reader.readAsText(file)

      reader.onload = () => {
        const result = reader.result
        console.log(Encoding.detect(result))
        const shiftJisCodeList = Encoding.convert(result, {
          to: 'UTF8',
          from: 'UNICODE',
          type: 'string'
        })
        console.log(shiftJisCodeList.toString())
        const lines = shiftJisCodeList.split('\n')
        console.log(lines)
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
        vm.data = linesArr
      }
    }
  }
}
</script>
