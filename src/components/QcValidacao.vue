<template>
  <div>
    <v-card>
      <v-card-title class="headline">
        Responda as seguintes perguntas para validar
      </v-card-title>
      <v-card-text>
        <template>
          <v-form ref="form" v-model="valid" lazy-validation>
            <div
              v-for="(item, index) in arrayPerguntas.perguntas"
              :key="'pergunta' + index"
            >
              <label v-if="item.valido!=false">{{ item.pergunta }}</label>
              <div v-if="item.pergunta.includes('carteirinha')&&item.valido!=false">
                <v-text-field
                  mask="99999999"
                  maxlength="8"
                  v-model="item.resposta"
                  placeholder="ex: 12345678"
                >
                </v-text-field>
              </div>
              <div v-if="item.pergunta.includes('CPF')&&item.valido!=false">
                <v-text-field
                  v-model="item.resposta"
                  mask="###.###.###-##"
                  placeholder="ex: 123.456.789-00"
                >
                </v-text-field>
              </div>
              <div v-else-if="item.pergunta.includes('nascimento')&&item.valido!=false">
                <v-text-field
                  v-model="item.resposta"
                  mask="##/##/####"
                  placeholder="dd/mm/aaaa"
                >
                </v-text-field>
              </div>
              <div v-else-if="item.pergunta.includes('UF')&&item.valido!=false">
                <v-text-field
                  v-model="item.resposta"
                  maxlength="2"
                  mask="AA"
                  placeholder="Ex: SP"
                >
                </v-text-field>
              </div>
              <div v-else-if="item.pergunta.includes('beneficiários')&&item.valido!=false">
                <v-text-field v-model="item.resposta" placeholder="Ex: 1">
                </v-text-field>
              </div>
              <div v-else-if="item.pergunta.includes('telefone')&&item.valido!=false">
                <v-text-field 
                v-model="item.resposta"     
                v-mask-phone.br
                placeholder="ex: (11) 9 1234-5678"           
                >                  
                </v-text-field>
              </div>
              <div v-else-if="item.pergunta.includes('e-mail')&&item.valido!=false">
                <v-text-field
                  v-model="item.resposta"
                >
                </v-text-field>
              </div>
              <div v-else-if="item.valido!=false">
                <v-text-field v-model="item.resposta"> </v-text-field>
              </div>
            </div>
          </v-form>
        </template>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
          <v-btn
            class="btn_blue_fonte_16_branco"
            text
            :loading="loading4"
            :disabled="loading4||validacaoFalha"
            @click="validarRespostas()"
            >Finalizar
          </v-btn>
      </v-card-actions>
    </v-card>
    <!-- Msg Aviso de Alerta -->
    <!-- <v-snackbar v-model="snackbarAlerta" color="blue darken-4" :auto-height="true" :timeout="timeout">
      <span class="title">{{ msgSnackAlerta}}</span>
      <v-btn color="white" flat @click="snackbarAlerta = false">Fechar</v-btn>
    </v-snackbar> -->

    <!-- Msg Aviso de Erro -->
    <v-snackbar v-model="snackbarErro" color="red" :auto-height="true" :timeout="timeout">
      <span class="title">{{ msgSnackErro}}</span>
      <v-btn color="white" flat @click="snackbarErro = false">Fechar</v-btn>
    </v-snackbar>

    <!-- Msg Sucesso -->
    <v-snackbar v-model="snackbarSucesso" color="green" :auto-height="true" :timeout="timeout">
      <span class="title">{{ msgSnackSucesso }}</span>
      <v-btn color="white" flat @click="snackbarSucesso = false">Fechar</v-btn>
    </v-snackbar>
  </div>
</template>

<script>
const uuidv4 = require("uuid/v4")

export default {
    props: [
    "respostas_",
    "server_core",
    "token",
    "token_api_perguntas",
    "id_pessoa",
    "matricula",
    "refresh_token",
    "login",
  ],
  data() {
    return { 
      timeout : 3000,
      snackbarErro: null,
      msgSnackErro: "",
      snackbarSucesso: null,
      msgSnackSucesso: '',
      arrayPerguntas: [],
      loading4: false,
      validacaoFalha: false,
      valid: true,
      emailRules: [(v) => /.+@.+\..+/.test(v) || "E-mail precisa ser válido"],
    };
  },
  async created(){
    await this.listarPerguntasRespostas();
  },
  methods: {
    async listarPerguntasRespostas() {
      // let link = `${this.server_core}/api-perguntas-acesso/buscar-perguntas?matricula=${this.matricula}&api-key=${this.token_api_perguntas}&ehMeioContato=true`
      let link = `http://localhost:19143/api-perguntas-acesso/buscar-perguntas-limitado?matricula=${this.matricula}&api-key=${this.token_api_perguntas}&ehMeioContato=true`
      try {
        fetch(link, {
          method: "get",
          headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
            token: this.token,
            refresh_token: this.refresh_token,
          },
        }).then((dataWrappedByPromise) => dataWrappedByPromise.json()).then((ret) => {
          this.arrayPerguntas = ret;
          console.log(ret, "ret");
          this.isEditing = false;
        })
      } catch (error) {
        this.snackbarErro = true;
        this.msgSnackErro = "Erro ao listar as perguntas!"
      }
    },
    async validarRespostas() {
      if(this.$refs.form.validate()){
        // this.$emit("solicitar-protocolo", async (numeroProtocolo) => {
          this.loading4 = true;
          // let link = `${this.server_core}/api-perguntas-acesso/validar-perguntas-retorno?api-key=${this.token_api_perguntas}`
          let link = `http://localhost:19143/api-perguntas-acesso/validar-perguntas-retorno?api-key=${this.token_api_perguntas}`

          for (let item of this.arrayPerguntas.perguntas) {
            if (
              item.pergunta.includes("mãe") ||
              item.pergunta.includes("cônjuge")
            ) {
              item.resposta = item.resposta.split(" ")[0];
            }
          }

          let content = {
            // numeroProtocolo: numeroProtocolo,
            matricula: this.matricula,
            perguntasHash: this.arrayPerguntas.perguntasHash,
            perguntas: this.arrayPerguntas.perguntas,
            login : this.login
          };

          try {
            await fetch(link, {
              method: "post",
              headers: {
                "Content-Type": "application/json",
                Accept: "application/json",
                token: this.token,
                refresh_token: this.refresh_token,
              },
              body: JSON.stringify(content),
            })
              .then(async (dataWrappedByPromise) => dataWrappedByPromise.json())

              .then(async (ret) => {
                let respostasValidadas = ret.valido ? true : false;
                this.validacaoFalha = ret.qtdErradas >= 3
                if (!respostasValidadas) {
                  this.snackbarErro = true;
                  this.msgSnackErro = ret.msg;
                  this.arrayPerguntas = ret.perguntas;
                  // this.$emit("alterar-permissao-edicao", {matricula: this.matricula ,validada: false});
                  // await this.listarPerguntasRespostas()
                } else {
                  // this.$emit("alterar-permissao-edicao", {matricula: this.matricula ,validada: true});
                  // this.fecharModal()
                  this.snackbarSucesso = true;
                  this.msgSnackSucesso = ret.msg
                  await this.listarPerguntasRespostas();
                }
              });
          } catch (error) {
            this.snackbarErro = true;
            this.msgSnackErro = "Erro ao tentar validar as perguntas!"
          }
          this.loading4 = false;
        // })
      }     
    },
  },
}
</script>