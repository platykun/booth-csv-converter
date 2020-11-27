<template>
  <v-container>
    <v-row>
      <input type="file" @change="loadCsvFile">
      <v-spacer/>
      <v-btn color="primary" @click="downloadCSV">
        csvダウンロード
      </v-btn>
    </v-row>

    <p>{{ message }}</p>
    <v-row>
      <v-col cols="10">
        <v-data-table
          :headers="header"
          :items="data"
        >
          <template v-slot:item="{ item }">
            <tr>
              <td v-for="(v, i) in item" :key="i">{{ v }}</td>
            </tr>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import * as parseCsv from 'csv-parse/lib/sync'
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
        console.log(shiftJisCodeList)
        const parsedCsv = parseCsv(shiftJisCodeList.toString(), {
          bom: true,
          relax_column_count: true
        })

        const header = []
        for (const h of parsedCsv[1]) {
          const hs = h.split('/')
          hs.forEach(splitedHeader => header.push({
            text: splitedHeader,
            value: splitedHeader
          })
          )
        }

        vm.header = header
        // TODOヌルポの例外処理は一旦省略

        const regex = new RegExp(/商品ID/)
        const body = []
        for (const row of parsedCsv.splice(2)) {
          const slicedRow = []
          const productsInfo = []

          for (const cell of row) {
            if (!regex.test(cell)) {
              slicedRow.push(cell)
            } else {
              // 分割対象となるcell
              // 以下のような値が入ってくるはずである
              // eslint-disable-next-line max-len
              // 商品ID : 1234567 / 数量 : 1 / ミラクル！フルーツTシャツ 商品ID : 2222222 / 数量 : 1 / 東風谷早苗 御朱印帳 商品ID : 333333 / 数量 : 1 / 東風谷早苗サイン入り！大幣
              for (const product of cell.split('\n')) {
                const productInfo = []
                // 以下のような値が入るはず
                // 商品ID : 1234567 / 数量 : 1 / ミラクル！フルーツTシャツ
                for (const productColumn of product.split('/')) {
                  // 以下のような値
                  // 商品ID : 1234567
                  const replacedColumn = productColumn.replace(/^.*: /g, '')
                  productInfo.push(replacedColumn)
                }
                productsInfo.push(productInfo)
              }
            }
          }
          console.log(slicedRow)
          console.log(productsInfo)

          for (const productInfo of productsInfo) {
            const row = JSON.parse(JSON.stringify(slicedRow))
            productInfo.forEach(p => row.push(p))
            body.push(row)
          }
        }
        vm.data = body
      }
      reader.readAsArrayBuffer(file)
    },
    downloadCSV () {
      let csv = '\uFEFF' + '品名,価格\n'
      this.data.forEach(el => {
        const line = el.name + ',' + el.price + '\n'
        csv += line
      })
      const blob = new Blob([csv], { type: 'text/csv' })
      const link = document.createElement('a')
      link.href = window.URL.createObjectURL(blob)
      link.download = 'Result.csv'
      link.click()
    }
  }
}
</script>
