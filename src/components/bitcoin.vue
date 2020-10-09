<template>
  <v-container>
    <v-row align="center" justify="space-around" class="pt-5 btn-style">
      <v-btn depressed :disabled="disabled" color="success" @click="start">Старт</v-btn>
      <v-btn depressed :disabled="disabled" color="primary" @click="reset">Сброс</v-btn>
      <v-btn depressed :disabled="disabled" color="error" @click="stop" >Стоп</v-btn>
    </v-row>

    <v-data-table :headers="headers" :items="store" item-key="name" class="elevation-1" :search="search">
      <template v-slot:item.actions="{ item }">
        <v-btn x-small color="secondary" dark @click="details(item)">Детали</v-btn>
      </template>

      <template v-slot:top>
        <v-text-field v-model="search" label="Поиск" class="mx-4"></v-text-field>
      </template>

    </v-data-table>

    <v-row no-gutters class="pa-5">
      <v-col cols="12" sm="4">
        <v-toolbar-title>Транзакций {{ store.length }}</v-toolbar-title>
      </v-col>
      <v-col cols="12" sm="4">
        <v-toolbar-title >Общая входная сумма {{ inputSum }}</v-toolbar-title>
      </v-col>
      <v-col cols="12" sm="4">
        <v-toolbar-title>Общая выходная сумма {{ outSum }}</v-toolbar-title>
      </v-col>
    </v-row>

    <v-dialog v-model="dialog" width="500">

      <v-card>
        <v-card-title class="headline grey lighten-2">JSON</v-card-title>

        <v-card-text>{{ dialogData }}</v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="dialog = false">Закрыть</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>



<script>
  export default {
    name: 'bitcoin',

    mounted() {
      this.ws = new WebSocket(this.src)
      this.ws.onmessage = event => this.add(event)
      this.ws.onopen = () => { this.disabled = false }
      this.ws.onclose = () => { this.disabled = true }
    },

    beforeDestroy() {
      this.ws.close()
      this.ws = null
    },

    data () {
      return {
        src: 'wss://ws.blockchain.info/inv',
        ws: null,
        disabled: true,
        dialog: false,
        dialogData: '',
        search: '',
        store: []
      }
    },
    computed: {
      inputSum () {
        return this.store.reduce((acc, i) => acc + (Number(i.inputs_value) || 0), 0)
      },
      outSum () {
        return this.store.reduce((acc, i) => acc + (Number(i.out_value) || 0), 0)
      },
      headers () {
        return [
          { text: 'Время', value: 'time' },
          { text: 'Хеш', value: 'hash' },
          { text: 'От кого', value: 'inputs_addr'},
          { text: 'Входная сумма', value: 'inputs_value'},
          { text: 'Кому', value: 'out_addr'},
          { text: 'Выходная сумма', value: 'out_value'},
          { text: 'JSON', value: 'actions', sortable: false },
        ]
      },
    },
    methods: {
      start () {
        this.ws.send('{"op":"unconfirmed_sub"}')
      },
      stop () {
        this.ws.send('{"op":"unconfirmed_unsub"}')
      },
      reset () {
        this.store = []
      },
      add (event) {
        let data = JSON.parse(event.data)
        this.store.push({
          time: data.x.time,
          hash: data.x.hash,
          inputs_addr: (data?.x?.inputs[0] || {}).prev_out?.addr,
          inputs_value: (data?.x?.inputs[0] || {}).prev_out?.value,
          out_addr: (data?.x?.out[0] || {}).addr,
          out_value: (data?.x?.out[0] || {}).value,
          data: event.data
        })
      },
      details (e) {
        this.dialogData = e.data
        this.dialog = true
      }
    }
  }
</script>
