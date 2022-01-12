<template>  
  <v-app class="white">
    <v-content>       
      <v-tooltip bottom class="ml-2">
        <template v-slot:activator="{ on }">
          <v-btn round v-on="on" @click="modalOpen = true">
            <v-icon>info</v-icon>
          </v-btn>
        </template>
        <span>Abrir Modal</span>
      </v-tooltip>
      <div class="text-center">
        <v-dialog v-model="modalOpen" width="500">
          <v-card>
            <v-card-title class="headline grey lighten-2">
              Titulo
            </v-card-title>
            <v-layout wrap pa-3>
              <v-flex xs12 sm6 d-flex>
                <v-select
                  :items="listaTemplates"
                  label="Solo field"
                  solo
                ></v-select>
              </v-flex>
            </v-layout>
            <v-divider></v-divider>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="success" text @click="modalOpen = false">
                Confirmar
              </v-btn>
              <v-btn color="error" text @click="modalOpen = false">
                Cancelar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </div>
    </v-content>
  </v-app>  
</template>

<script>
import QcValidacao from './QcValidacao'

export default {
  components: {
    QcValidacao: QcValidacao
  },
  data() {
    return {
      modalOpen: false,
      listaTemplates: ['Foo', 'Bar', 'Fizz', 'Buzz']
    }
  },
  props: [
    "nome",
    'server_core'
  ],
  methods: {
    async listarPerguntasRespostas() {
      let link = `${this.server_core}/api-base/criar-template`
      try {
        fetch(link, {
          method: "get",
          headers: {
            "Content-Type": "application/json",
            Accept: "application/json"
          },
        }).then((ret) => {
          this.listaTemplates = ret
        })
      } catch (error) {
        this.snackbarErro = true
        this.msgSnackErro = "Erro ao listar as perguntas!"
      }
    }
  }
}
</script>

<style>
  @import '../../src/assets/css/vuetify.min.css';
  @import '../../node_modules/material-design-icons-iconfont/dist/material-design-icons.css'; 
  @import '../../node_modules/@fortawesome/fontawesome-free/css/svg-with-js.css';
  @import '../../node_modules/font-awesome/css/font-awesome.min.css';
  @import '../../src/assets/css/qc-wc-default.css';
</style>
